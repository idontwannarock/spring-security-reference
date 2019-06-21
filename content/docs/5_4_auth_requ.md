---
title: "5.4. 設定授權請求"
---

# 5.4. 設定授權請求

在前面的範例中，我們只要求每個 URL 都需要使用者通過驗證，但我們也可以透過在 `http.authorizeRequests()` 方法中，使用其他方法串接來對 URL 做更細緻的設定，例如：

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()                                                     // 1.
			.antMatchers("/resources/**", "/signup", "/about").permitAll()       // 2.
			.antMatchers("/admin/**").hasRole("ADMIN")                           // 3.
			.antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')") // 4.
			.anyRequest().authenticated()                                        // 5.
			.and()
		// ...
		.formLogin();
}
```

1. 在 `http.authorizeRequests()` 後面串接的其他 matcher，會按照被列舉的順序比對
2. 同時宣告多種 URL 樣式，並允許所有使用者都有權限請求
3. 必須有 ADMIN 腳色權限的使用者才能對 /admin/ 開頭的 URL 做請求。注意使用 `hasRole()` 方法設定腳色的時候，腳色名的 "ROLE_" 前綴可以省略
4. 必須同時有 ADMIN 及 DBA 兩種腳色的權限才能對 /db/ 開頭的 URL 做請求
5. 任何不符合已經列舉的 URL 樣式，使用者都必須經過驗證