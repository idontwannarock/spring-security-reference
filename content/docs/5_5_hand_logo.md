---
title: "5.5. 登出處理"
type: docs
---

# 5.5. 登出處理

當使用了 `WebSecurityConfigurerAdapter` 就會自動開啟登出的功能，預設會用 `/logout` 路徑讓使用者登出，並執行以下的動作：

1. 註銷該 HTTP Session
2. 清除已設定的記得我驗證
3. 清空 `SecurityContextHolder`
4. 重新導向到 `/login?logout`

不過就如同登入設定，登出設定也有很多種選項可以依需求客製：

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		.logout()                                       // 1.
			.logoutUrl("/my/logout")                    // 2.
			.logoutSuccessUrl("/my/index")              // 3.
			.logoutSuccessHandler(logoutSuccessHandler) // 4.
			.invalidateHttpSession(true)                // 5.
			.addLogoutHandler(logoutHandler)            // 6.
			.deleteCookies(cookieNamesToClear)          // 7.
			.and()
		...
}
```

1. 提供登出功能的支援，當使用了 `WebSecurityConfigurerAdapter` 就會自動支援此功能
2. 設定觸發登出功能的 URL，預設為 `/logout`。注意如果開啟預設的 CSRF 保護機制，那此請求必須是 POST，詳細參考 [JavaDoc](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configurers/LogoutConfigurer.html#logoutUrl-java.lang.String-)
3. 登出後會重新導向的 URL，預設是 `/login?logout`，詳細參考 [JavaDoc](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configurers/LogoutConfigurer.html#logoutSuccessUrl-java.lang.String-)
4. 容許提供自訂的 `LogoutSuccessHandler`，如果有列出此項，則 `logoutSuccessUrl()` 的部分會被忽略，詳細參考 [JavaDoc](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configurers/LogoutConfigurer.html#logoutSuccessHandler-org.springframework.security.web.authentication.logout.LogoutSuccessHandler-)
5. 設定登出後是否要註銷 `HttpSession`，預設為 `true`，此項其實是在背後對 `SecurityContextLogoutHandler` 做設定，詳細參考 [JavaDoc](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configurers/LogoutConfigurer.html#invalidateHttpSession-boolean-)
6. 加入 `LogoutHandler`。預設會將 `SecurityContextLogoutHandler` 作為最後一個 `LogoutHandler` 加入
7. 設定成功登出後要刪除的 cookie 名稱，其實等於加入 `CookieClearingLogoutHandler`

> 登出當然也可以透過 XML 命名空間的方式做設定

為了讓登出順利運作，通常會加上 [`LogoutHandler`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/LogoutHandler.html) 跟 [`LogoutSuccessHandler`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/LogoutSuccessHandler.html) 的實作。在許多常見的情況，Spring Security 會在背後以 [流式接口(方法鍊式調用)](https://zh.wikipedia.org/wiki/%E6%B5%81%E5%BC%8F%E6%8E%A5%E5%8F%A3) 來加上這些 handler。

## 5.5.1. LogoutHandler

一般來說，[`LogoutHandler`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/LogoutHandler.html) 的實作包括那些可以參與登出處理的類別，這些類別通常都被調用來做某些必要的清除動作，因此他們不應該拋出例外。

Spring Security 提供許多不同的實作：

- [PersistentTokenBasedRememberMeServices](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/rememberme/PersistentTokenBasedRememberMeServices.html)
- [TokenBasedRememberMeServices](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/rememberme/TokenBasedRememberMeServices.html)
- [CookieClearingLogoutHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/CookieClearingLogoutHandler.html)
- [CsrfLogoutHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/csrf/CsrfLogoutHandler.html)
- [SecurityContextLogoutHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html)

細節請參考 _18.4. 記得我介面及實作_。

除了直接提供 `LogoutHandler` 的實作，流式接口在背後有提供對應 `LogoutHandler` 實作的捷徑方法，例如 `deleteCookies()` 方法允許你指定一個或多個 cookie 的名字，以在成功登出後移除它們，這個方法等於加入 `CookieClearingLogoutHandler`。

## 5.5.2. LogoutSuccessHandler

在 `LogoutFilter` 成功登出後，`LogoutSuccessHandler` 會被呼叫來處理例如重新導向到合適的目的地等動作。注意這個介面與 `LogoutHandler` 幾乎一樣，但是可能會拋出例外。

Spring Security 提供以下實作：

- [`SimpleUrlLogoutSuccessHandler`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/SimpleUrlLogoutSuccessHandler.html)
- [HttpStatusReturningLogoutSuccessHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/logout/HttpStatusReturningLogoutSuccessHandler.html)

如同前一個段落提到的，你不需要直接加上 `SimpleUrlLogoutSuccessHandler`，可以改用 `logoutSuccessUrl()`，這同樣會在背後設定好 `SimpleUrlLogoutSuccessHandler`，在登出動作發生時會導向到提供的 URL，預設是 `/login?logout`。

如果是在 REST API 的情境下，`HttpStatusReturningLogoutSuccessHandler` 就會很有趣唷，啾咪 ^.<

這個 `LogoutSuccessHandler` 允許在登出成功後，返回你提供的 HTTP 狀態碼，而不是轉向某個 URL，如果沒有特別設定，預設會返回狀態碼 200。

## 5.5.3. 其他跟 Logout 相關的參考

- 6.2.4. 登出處理
- 12.3.2. 測試登出
- [16.2.3. `HttpServletRequest.logout()`]({{< relref "/docs/16_2_serv_3_plus_inte.md#16-2-3-httpservletrequest-logout" >}})
- [18.4. 記得我介面及實作]({{< relref "/docs/18_4_reme_me_inte_and_impl.md" >}})
- 19.5.3. 登出及 CSRF 注意事項
- 34.3.2. 單一登出 (CAS 協議)
- 43.1.26 `<logout>` 命名空間標籤