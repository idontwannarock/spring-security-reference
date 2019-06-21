---
title: "5.8. 權限驗證"
---

# 5.8. 權限驗證

到目前為止，我們只看了如何進行最基本的驗證設定，接下來看一些更進階一些的選項。

## 5.8.1. In-Memory 驗證

前面已經看過設定單一使用者的 in-memory 驗證範例，接著來看如何設定多使用者：

```java
@Bean
public UserDetailsService userDetailsService() throws Exception {
	// 以確保密碼經過良好的編碼
	UserBuilder users = User.withDefaultPasswordEncoder();
	InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
	manager.createUser(users.username("user").password("password").roles("USER").build());
	manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
	return manager;
}
```

## 5.8.2. JDBC 驗證

這個版本有支援以 JDBC 為基礎的驗證機制，下面的範例假設你已經在應用程式中定義好一個 `DataSource`。[這裡](https://github.com/spring-projects/spring-security/tree/master/samples/javaconfig/jdbc) 也有一個使用 JDBC 基礎驗證的完整範例

```java
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	// 以確保密碼經過良好的編碼
	UserBuilder users = User.withDefaultPasswordEncoder();
	auth
		.jdbcAuthentication()
			.dataSource(dataSource)
			.withDefaultSchema()
			.withUser(users.username("user").password("password").roles("USER"))
			.withUser(users.username("admin").password("password").roles("USER","ADMIN"));
}
```

## 5.8.3. LDAP 驗證

這個版本有支援以 LDAP 為基礎的驗證機制，[這裡](https://github.com/spring-projects/spring-security/tree/master/samples/javaconfig/ldap) 有一個使用 LDAP 基礎驗證的完整範例

```java
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	auth
		.ldapAuthentication()
			.userDnPatterns("uid={0},ou=people")
			.groupSearchBase("ou=groups");
}
```

上面這個範例遵循了 LDIF 格式，並使用了一個嵌入的 Apache DS LDAP 實例

**users.ldif**

```
dn: ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: groups

dn: ou=people,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: people

dn: uid=admin,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Rod Johnson
sn: Johnson
uid: admin
userPassword: password

dn: uid=user,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Dianne Emu
sn: Emu
uid: user
userPassword: password

dn: cn=user,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: groupOfNames
cn: user
uniqueMember: uid=admin,ou=people,dc=springframework,dc=org
uniqueMember: uid=user,ou=people,dc=springframework,dc=org

dn: cn=admin,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: groupOfNames
cn: admin
uniqueMember: uid=admin,ou=people,dc=springframework,dc=org
```

## 5.8.4. `AuthenticationProvider`

可以透過將一個自訂 `AuthenticationProvider` 設定為一個 bean 來調整驗證機制。

以下範例假設 `SpringAuthenticationProvider` 實作了 `AuthenticationProvider`。

> 這方法只有當 `AuthenticationManagerBuilder` 還沒有被指定 provider 時才會被使用

```java
@Bean
public SpringAuthenticationProvider springAuthenticationProvider() {
	return new SpringAuthenticationProvider();
}
```

## 5.8.5. `UserDetailsService`

你也可以透過設定一個自訂 `UserDetailsService` 的 bean 來調整驗證機制。

以下範例假設 `SpringDataUserDetailsService` 實作了 `UserDetailsService`。

> 這方法只有當 `AuthenticationManagerBuilder` 還沒有被指定 provider，且還沒有定義任何的 `AuthenticationProviderBean` 時才會被使用

```java
@Bean
public SpringDataUserDetailsService springDataUserDetailsService() {
	return new SpringDataUserDetailsService();
}
```

你也可以透過定義一個 `PasswordEncoder` 的 bean 來設定密碼如何編碼。

例如以下定義一個 bean 來使用 bcrypt：

```java
@Bean
public BCryptPasswordEncoder passwordEncoder() {
	return new BCryptPasswordEncoder();
}
```