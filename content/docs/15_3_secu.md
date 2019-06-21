---
title: "15.3. `SecurityContextPersistenceFilter`"
type: docs
---

# 15.3. `SecurityContextPersistenceFilter`

我們在 [9.4.4. 在請求之間保存 `SecurityContext`]({{< relref "/docs/9_4_auth_in_web_app.md#9-4-4-在請求之間保存-securitycontext" >}}) 這個章節已經探討過這個非常重要的 filter 的目的，所以你也可以趁機重新複習一下那個部分。

首先我們來看一下如何對它做設定好讓 `FilterChainProxy` 可以使用，基本的設定只需要 bean：

```xml
<bean id="securityContextPersistenceFilter" class="org.springframework.security.web.context.SecurityContextPersistenceFilter"/>
```

如同我們之前看過的，這個 filter 主要有兩個任務：

- 在 HTTP 請求之間保存 `SecurityContext` 的內容
- 在一個請求結束的時候，清除 `SecurityContextHolder`

清除 `ThreadLocal` 保存的 context 是很關鍵的，因為它有可能會被放到 servlet 容器的執行續池，而且裡面還包含著使用者的安全資訊，然後這個執行續可能在後續的階段被用來執行其他行動，但是用錯誤的憑證。

## 15.3.1. `SecurityContextRepository`

從 Spring Security 3.0 開始，裝載及保存安全資訊的任務被委託給一個分離的介面：

```java
public interface SecurityContextRepository {

    SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder);

    void saveContext(SecurityContext context, HttpServletRequest request, HttpServletResponse response);
}
```

`HttpRequestResponseHolder` 只是一個用來裝著收到的請求及回應物件的容器，使得這個實作可以將它們換為封裝類別，返回的物件則會被傳給 filter 鍊。

`HttpSessionSecurityContextRepository` 是預設的實作，它會將 security context 存成一個 `HttpSession` 性質。而這個實作最重要的設定參數 `allowSessionCreation`，預設為 `true`，因此當這個類別需要存放一個通過驗證的使用者的 security context 時，可以產生一個 session 來保存。但只有當驗證動作有發生而且 security context 的內容有改變時才會建立 session。

如果你不想要建立 session，你可以將此屬性設定為 `false`：

```xml
<bean id="securityContextPersistenceFilter" class="org.springframework.security.web.context.SecurityContextPersistenceFilter">
    <property name='securityContextRepository'>
	    <bean class='org.springframework.security.web.context.HttpSessionSecurityContextRepository'>
	        <property name='allowSessionCreation' value='false' />
	    </bean>
    </property>
</bean>
```

或者你也可以使用 `NullSecurityContextRepository` 這個 null 物件的實作來避免 security context 被儲存，即使是請求過程中已經產生一個 session 也是一樣。