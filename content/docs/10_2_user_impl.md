---
title: "10.2. 實作 `UserDetailsService`"
type: docs
---

# 10.2. 實作 `UserDetailsService`

前面有提過多數 provider 都會利用 `UserDetails` 及 `UserDetailsService`。

`UserDetailsService` 介面預設有唯一的一個方法：

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

方法返回的 `UserDetails` 是一個提供 getter 的介面，提供非空值的驗證資訊，例如 username、password、granted authorities，以及使用者帳號是否啟用或停用等。

多數的驗證 provider 都會用到 `UserDetailsService`，即使驗證通過與否並沒有用到 username 及 password，可能只是為了其中的 `GrantedAuthority` 資訊，例如 LDAP、X.509 或 CAS 等，其他系統會負責實際去驗證憑證。

`UserDetailsService` 是很容易實作的，使用者應該可以很輕鬆的選用某種永續層策略來取得驗證資訊。

Spring Security 有提供了一些實用的基本實作，我們接著來看R。

## 10.2.1. In-Memory Authentication

`UserDetailsService` 的實作中去使用某種永續層引擎來取得資訊是很容易的，但其實很多應用程式並不需要搞得這麼複雜，特別是當你還在建立應用程式的雛型或剛開始整合 Spring Security 的時候，並不會特別想花時間去設定資料庫或真的去寫 `UserDetailsService` 的實作。

在這種情況下，可以簡單的使用 security 命名空間的 `user-service` 元素：

```xml
<user-service id="userDetailsService">
<!-- 密碼的前綴字 {noop} 是用來跟 DelegatingPasswordEncoder 說要使用 NoOpPasswordEncoder (也就是什麼事都不做的 PasswordEncoder)。這樣對於正式環境來說其實並不安全，只是讓讀取範本的時候更容易一點。一般來說密碼需要被 hash 跟 BCrypt -->
    <user name="jimi" password="{noop}jimispassword" authorities="ROLE_USER, ROLE_ADMIN" />
    <user name="bob" password="{noop}bobspassword" authorities="ROLE_USER" />
</user-service>
```

也支援使用外部 properties 檔做設定：

```xml
<user-service id="userDetailsService" properties="users.properties"/>
```

這種 properties 檔應該包含以下格式的資訊：

```
username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]
```

例如：

```
jimi=jimispassword,ROLE_USER,ROLE_ADMIN,enabled
bob=bobspassword,ROLE_USER,enabled
```

## 10.2.2. `JdbcDaoImpl`

Spring Security 也支援以 JDBC 取得驗證資訊的 `UserDetailsService` 實作，這樣可以避免使用某種 ORM 工具，但只是為了存放使用者資訊的複雜度。

當然如果你的應用程式本來就使用某種 ORM 工具，那你也可以考慮自己實作 `UserDetailsService` 去使用可能本來就準備好的映射檔。

回到 `JdbcDaoImpl`，設定檔的範例可能生做這款：

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
    <property name="url" value="jdbc:hsqldb:hsql://localhost:9001"/>
    <property name="username" value="sa"/>
    <property name="password" value=""/>
</bean>

<bean id="userDetailsService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

修改上面的 `DriverManagerDataSource` 就可以改用不同的 RDBMS，也可以使用從 JNDI 取得的 datasource。

### 權限群組

`JdbcDaoImpl` 預設會取得單一使用者的所有權限，並假定權限是直接對應到使用者。

另一種作法是將各種權限區分為群組，再將群組對應到使用者，這部分可以參考 [`JdbcDaoImpl` 的 API](https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/api/org/springframework/security/core/userdetails/jdbc/JdbcDaoImpl.html)。