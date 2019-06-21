---
title: "5.3. Java 設定及表單登入"
type: docs
---

# 5.3. Java 設定及表單登入

接著，既然我們並沒有提到任何 HTML 或 JSP 檔，你可能會懷疑系統提示要登入的表格從哪裡來。

雖然預設設定並沒有特別為登入頁面設定一個 URL，Spring Security 還是會基於一些預設被啟用的功能，自動產生一個頁面、使該頁面對處理提交登入的 URL 使用標準設定、自動設定使用者登入後會轉向的預設目標 URL 等等。

不過自動產生登入畫面雖然讓建立跟快速運作起來很方便，大多數的應用程式還是會希望提供特定的登入頁面。我們可以透過以下設定來達成：

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.anyRequest().authenticated()
			.and()
		.formLogin()
			.loginPage("/login") // 1.
			.permitAll();        // 2.
}
```

1. 此處的設定表示登入頁面的實體位置
2. 讓所有使用端都能取得登入頁面

基於目前設定的 JSP 登入頁面範例如下：

```jsp
<c:url value="/login" var="loginUrl"/>
<form action="${loginUrl}" method="post"> <!-- 1. -->
	<c:if test="${param.error != null}">  <!-- 2. -->
		<p>
			Invalid username and password.
		</p>
	</c:if>
	<c:if test="${param.logout != null}"> <!-- 3. -->
		<p>
			You have been logged out.
		</p>
	</c:if>
	<p>
		<label for="username">Username</label>
		<input type="text" id="username" name="username"/> <!-- 4. -->
	</p>
	<p>
		<label for="password">Password</label>
		<input type="password" id="password" name="password"/> <!-- 5. -->
	</p>
	<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/> <!-- 6. -->
	<button type="submit" class="btn">Log in</button>
</form>
```

1. POST 請求到 `/login` 以驗證使用者
2. 如果查詢參數 `error` 存在，驗證會被嘗試並失敗
3. 如果查詢參數 `logout` 存在，則使用者會被成功登出
4. HTTP 主體中必須存在以 username 為名稱的使用者名稱參數
5. HTTP 主體中必須存在以 password 為名稱的密碼參數
6. HTTP 主體中必須包含 CSRF Token