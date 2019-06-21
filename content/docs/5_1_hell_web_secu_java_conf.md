---
title: "5.1. 初到貴寶地來用個 Java 設定"
---

# 5.1. 初到貴寶地來用個 Java 設定

首先要建立 Java 設定類，這個設定類會建立一個被稱為 `springSecurityFilterChain` 的 Servlet Filter，負責應用程式的所有安全控制，例如保護應用程式的 URL、驗證提交的 username 及 passwords、重新導向登入表格等。

以下是一個最基本的 Java 設定類範例：

```java
import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.authentication.builders.*;
import org.springframework.security.config.annotation.web.configuration.*;

@EnableWebSecurity
public class WebSecurityConfig implements WebMvcConfigurer {

	@Bean
	public UserDetailsService userDetailsService() throws Exception {
		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(User.withDefaultPasswordEncoder().username("user").password("password").roles("USER").build());
		return manager;
	}
}
```

> 重點是 `@EnableWebSecurity` 這個註解跟實作 `WebMvcConfigurer` 這個介面

我們並沒有做什麼設定，但它就會幫我們做掉很多事：

- 對應用程式所有 URL 作驗證
- 自動產生登入表格
- 允許填入表格名稱為 _user_、密碼為 _password_ 的使用者進行驗證
- 允許使用者登出
- 阻絕 [CSRF](https://zh.wikipedia.org/zh-tw/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0) 攻擊
- 防止 [Session Fixation](https://en.wikipedia.org/wiki/Session_fixation)
- 整合 Security 標頭
    - 使用 [HSTS](https://zh.wikipedia.org/zh-tw/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8) 保護請求
    - [X-Content-Type-Options](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622941(v=vs.85)) 整合
    - 緩存控制 (後續可以在你的應用程式中改為允許緩存靜態資源)
    - [X-XSS-Protection](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/dd565647(v=vs.85)) 整合
    - X-Frame-Options 整合以避免 [Clickjacking](https://zh.wikipedia.org/zh-tw/%E7%82%B9%E5%87%BB%E5%8A%AB%E6%8C%81)
- 整合下列 Servlet API 方法
    - [`HttpServletRequest#getRemoteUser()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getRemoteUser())
    - [`HttpServletRequest.html#getUserPrincipal()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getUserPrincipal())
    - [`HttpServletRequest.html#isUserInRole(java.lang.String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#isUserInRole(java.lang.String))
    - [`HttpServletRequest.html#login(java.lang.String, java.lang.String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login(java.lang.String,%20java.lang.String))
    - [`HttpServletRequest.html#logout()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout())

## 5.1.1. `AbstractSecurityWebApplicationInitializer`

下一步則是將 `springSecurityFilterChain` 註冊到 war 檔。

在 Servlet 3.0 以上的環境，這步驟可以用 Java 設定透過 [Spring 對 WebApplicationInitializer 的支援](https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-container-config) 來完成。

當然 Spring Security 也提供了一個類別 `AbstractSecurityWebApplicationInitializer` 以幫你註冊 `springSecurityFilterChain`。

但我們怎麼使用 `AbstractSecurityWebApplicationInitializer` 取決於我們的應用程式是否已經使用 Spring 或者 Spring Security 是應用程式中唯一的 Spring 組件。

## 5.1.2. `AbstractSecurityWebApplicationInitializer` 與非 Spring 應用

什麼年代了當然是直接用 Spring 啊？跳過。

## 5.1.3. `AbstractSecurityWebApplicationInitializer` 與 Spring MVC 應用

如果我們的應用程式已經使用了 Spring，那我們可能早就有 `WebApplicationInitializer` 來載入 Spring 的設定，所以我們應該要在現存的 `ApplicationContext` 中註冊 Spring Security。

例如在 Spring MVC 中，我們的 `SecurityWebApplicationInitializer` 可能會長這樣：

```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

}
```

這樣只是將 `springSecurityFilterChain` 註冊為攔截應用程式所有的 URL，接著我們應該要確保在現存的 `ApplicationInitializer` 中會載入 `WebSecurityConfig`。

例如在 Spring MVC 中我們會使用 `getRootConfigClasses()` 來作載入：

```java
public class MvcWebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[] { WebSecurityConfig.class };
	}
}
```