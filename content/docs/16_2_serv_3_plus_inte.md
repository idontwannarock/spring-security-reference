---
title: "16.2. Servlet 3+ 整合"
type: docs
---

# 16.2. Servlet 3+ 整合

這個段落會說明 Spring Security 與 Servlet 3 方法的整合。

## 16.2.1. `HttpServletRequest.authenticate(HttpServletRequest,HttpServletResponse)`

[`HttpServletRequest.authenticate(HttpServletRequest,HttpServletResponse)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#authenticate%28javax.servlet.http.HttpServletResponse%29) 方法可以用來確保使用者經過驗證。

如果還沒驗證，則設定好的 `AuthenticationEntryPoint` 會被觸發來要求使用者作驗證，例如轉導到登入頁面。

## 16.2.2. `HttpServletRequest.login(String,String)`

[`HttpServletRequest.login(String,String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#login%28java.lang.String,%20java.lang.String%29) 方法可以被用來使用當前的 `AuthenticationManager` 來驗證使用者。

例如以下範例會試圖以使用者名稱 `user` 及密碼 `password` 來作驗證：

```java
try {
    httpServletRequest.login("user","password");
} catch (ServletException e) {
    // fail to authenticate
}
```

> 如果你希望 Spring Security 來處理失敗的驗證，則不需要 catch ServletException

## 16.2.3. `HttpServletRequest.logout()`

[`HttpServletRequest.logout()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#logout%28%29) 方法可以用來登出目前的使用者。

通常這代表 `SecurityContextHolder` 會被清空，`HttpSession` 會被廢止，任何「記得我」認證都會被清除等等。但是設定好的 `LogoutHandler` 實作也可以能因為你的 Spring Security 設定而有所不同。所以在 `HttpServletRequest.logout()` 被觸發後，你還能控制回應寫出是非常重要的，例如通常會轉導到歡迎頁面。

## 16.2.4. `AsyncContext.start(Runnable)`

[`AsyncContext.start(Runnable)`](https://docs.oracle.com/javaee/6/api/javax/servlet/AsyncContext.html#start%28java.lang.Runnable%29) 方法是用來確保你的憑證會被傳遞到新的 `Thread`。

使用 Spring Security 對併發的支援，Spring Security 會覆蓋 `AsyncContext.start(Runnable)` 方法來確保處理 `Runnable` 時使用的是當前的 `SecurityContext`。

例如以下範例會回應目前使用者的 `Authentication`：

```java
final AsyncContext async = httpServletRequest.startAsync();
async.start(new Runnable() {
	public void run() {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		try {
			final HttpServletResponse asyncResponse = (HttpServletResponse) async.getResponse();
			asyncResponse.setStatus(HttpServletResponse.SC_OK);
			asyncResponse.getWriter().write(String.valueOf(authentication));
			async.complete();
		} catch(Exception e) {
			throw new RuntimeException(e);
		}
	}
});
```

## 16.2.5. 非同步 Servlet 支援

如果你是使用 Java 設定，你可以直接使用非同步 Servlet 支援。但如果你是使用 XML，那還有一些必要的設定要做。

第一步是確保你將你的 web.xml 更新到最起碼 3.0 的 schema，如以下：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

</web-app>
```

然後要確保你的 `springSecurityFilterChain` 被設定為處理非同步請求。

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <async-supported>true</async-supported>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>ASYNC</dispatcher>
</filter-mapping>
```

就這樣！現在 Spring Security 會確保你的 `SecurityContext` 也會傳遞到非同步的請求中。

那它是怎麼進行的？如果你沒有特別想知道，你完全可以跳過這段落剩下的部分。

這功能大部分都是建立在 Servlet 規格上，但 Spring Security 有做了一些些調整來確保適當的處理非同步請求。

在 Spring Security 3.2 以前，當 `HttpServletResponse` 被提交時，`SecurityContextHolder` 內的 `SecurityContext` 會立刻被自動保存。這在非同步的環境下會造成一些問題。

例如，考慮以下狀況：

```java
httpServletRequest.startAsync();
new Thread("AsyncThread") {
	@Override
	public void run() {
		try {
			// Do work
			TimeUnit.SECONDS.sleep(1);

			// Write to and commit the httpServletResponse
			httpServletResponse.getOutputStream().flush();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}.start();
```

這個狀況的問題是 Spring Security 並不知道這個 `Thread`，所以 `SecurityContext` 並不會傳遞給它，也就是說當我們提交 `HttpServletResponse` 的時候，`SecurityContext` 並不存在。所以當提交 `HttpServletResponse` 並且要自動保存 `SecurityContext` 的時候，就會失去已經登入的使用者。

從 3.2 版開始，當提交 `HttpServletResponse` 並觸發 `HttpServletRequest.startAsync()` 時，Spring Security 就不再會自動保存 `SecurityContext`。