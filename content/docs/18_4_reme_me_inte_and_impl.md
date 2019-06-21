---
title: "18.4. 記得我介面及實作"
---

# 18.4. 記得我介面及實作

記得我功能是與 `UsernamePasswordAuthenticationFilter` 一起使用，並且是透過父類別 `AbstractAuthenticationProcessingFilter` 裡的掛勾來實作。它也會在 `BasicAuthenticationFilter` 內部使用。這個掛勾會在合適的時間點觸發有實作的 `RememberMeServices`。此介面長這個樣子：

```java
Authentication autoLogin(HttpServletRequest request, HttpServletResponse response);

void loginFail(HttpServletRequest request, HttpServletResponse response);

void loginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication successfulAuthentication);
```

要了解每個方法做些什麼，請參考 Javadoc 裡面更完整的討論，但注意在這個階段，`AbstractAuthenticationProcessingFilter` 只會呼叫 `loginFail()` 及 `loginSuccess()` 兩個方法。

`autoLogin()` 方法則是當 `SecurityContextHolder` 並不持有一個 `Authentication` 的時候，由 `RememberMeAuthenticationFilter` 來呼叫，所以這個介面會提供對於驗證相關事件足夠提醒的記得我功能底層實作，並且當一個網路請求可能包含一個 cookie 且希望被記得的時候，代理給該實作。這個設計容許任意數量的記得我實作策略。

我們在前面已經知道 Spring Security 提供兩種實作，我們會依序來看。

## 18.4.1. `TokenBasedRememberMeServices`

這個實作支援在 [18.2. 簡單雜湊 Token 方式]({{< relref "/docs/18_2_simp_hash_base_toke_appr.md" >}}) 裡描述的簡單方式。在 `RememberMeAuthenticationProvider` 處理的過程中，會調用 `TokenBasedRememberMeServices` 來產生一個 `RememberMeAuthenticationToken`。一個 `key` 會在這個驗證提供者及 `TokenBasedRememberMeServices` 之間傳遞。

另外，為了比對 token 的簽名，`TokenBasedRememberMeServices` 還需要一個 UserDetailsService 來取得使用者名稱及密碼，並產生包含正確 `GrantedAuthority` 的 `RememberMeAuthenticationToken`。

如果使用者要求的話，應用程式應該提供某種形式的登出指令來使 cookie 失效。

`TokenBasedRememberMeServices` 也實作 Spring Security 的 `LogoutHandler` 介面，所以可以與 `LogoutFilter` 一起使用來自動清除 cookie。

要使用記得我功能，需要在 application context 設定這些 bean：

```xml
<bean id="rememberMeFilter" class="org.springframework.security.web.authentication.rememberme.RememberMeAuthenticationFilter">
    <property name="rememberMeServices" ref="rememberMeServices"/>
    <property name="authenticationManager" ref="theAuthenticationManager" />
</bean>

<bean id="rememberMeServices" class="org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices">
    <property name="userDetailsService" ref="myUserDetailsService"/>
    <property name="key" value="springRocks"/>
</bean>

<bean id="rememberMeAuthenticationProvider" class="org.springframework.security.authentication.RememberMeAuthenticationProvider">
    <property name="key" value="springRocks"/>
</bean>
```

然後別忘記將妳的 `RememberMeService` 實作加入到你的 `UsernamePasswordAuthenticationFilter.setRememberMeServices()` 屬性中，並將 `RememberMeAuthenticationProvider` 加入 `AuthenticationManager.setProviders()` 的清單中，還有將 `RememberMeAuthenticationFilter` 放到你的 `FilterChainProxy` 裡面，通常是緊接著 `UsernamePasswordAuthenticationFilter`。

## 18.4.2. `PersistentTokenBasedRememberMeServices`

這個類別使用方式與 `TokenBasedRememberMeServices` 相同，但需要額外與一個 `PersistentTokenRepository` 一併做設定以保存 token。

有兩個標準的實作：

- `InMemoryTokenRepositoryImpl`：通常是測試用
- `JdbcTokenRepositoryImpl`：將 token 保存在 DB

DB Schema 在前面的 [18.3. 持久化 Token 方式]({{< relref "/docs/18_3_pers_toke_appr.md" >}}) 裡面有描述。