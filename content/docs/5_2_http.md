---
title: "5.2. `HttpSecurity`"
type: docs
---

# 5.2. `HttpSecurity`

到目前為止，我們的 `WebSecurityConfig` 只包含了如何驗證使用者的資訊，但 Spring Security 要怎麼知道我們要求所有者用者都需要進行驗證？Spring Security 如何知道我們要使用表單驗證？

這是因為 `WebSecurityConfigurerAdapter` 在 `configure(HttpSecurity http)` 方法中有提供一些預設的設定如下：

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.anyRequest().authenticated()
			.and()
		.formLogin()
			.and()
		.httpBasic();
}
```

以上的預設設定包括：

- 確保所有對應用程式的請求都需要使用者通過驗證
- 允許使用者使用表單驗證
- 允許使用者使用 HTTP Basic 驗證

這跟 XML 命名空間方式的設定很像：

```xml
<http>
	<intercept-url pattern="/**" access="authenticated"/>
	<form-login />
	<http-basic />
</http>
```

Java 設定方式的 `and()` 等於 XML 的結束標籤，這樣我們就可以繼續對物件作設定。

但是 Java 設定方式有不同的預設 URL 跟參數，在自訂登入頁面的時候要注意這一點，這使得 URL 更貼近 RESTful 風格；另外，這樣讓我們在使用 Spring Security 上不會那麼明顯，以避免 [資訊洩漏 Information Leak](https://www.owasp.org/index.php/Information_Leak_(information_disclosure)) 的問題。