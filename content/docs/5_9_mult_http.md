---
title: "5.9. 多重 `HttpSecurity`"
---

# 5.9. 多重 `HttpSecurity`

我們可以設定多個 `HttpSecurity` 實例，如同在命名空間用多個 `<http>` 作設定一樣。關鍵是要在多處繼承 `WebSecurityConfigurationAdapter`。

以下範例示範如何對 `/api/` 開頭的 URL 作不同的設定：

```java
@EnableWebSecurity
public class MultiHttpSecurityConfig {
	@Bean                                                             // 1.
	public UserDetailsService userDetailsService() throws Exception {
		// 確保密碼有良好的編碼
		UserBuilder users = User.withDefaultPasswordEncoder();
		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(users.username("user").password("password").roles("USER").build());
		manager.createUser(users.username("admin").password("password").roles("USER","ADMIN").build());
		return manager;
	}

	@Configuration
	@Order(1)                                                        // 2.
	public static class ApiWebSecurityConfigurationAdapter extends WebSecurityConfigurerAdapter {
		protected void configure(HttpSecurity http) throws Exception {
			http
				.antMatcher("/api/**")                               // 3.
				.authorizeRequests()
					.anyRequest().hasRole("ADMIN")
					.and()
				.httpBasic();
		}
	}

	@Configuration                                                   // 4.
	public static class FormLoginWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeRequests()
					.anyRequest().authenticated()
					.and()
				.formLogin();
		}
	}
}
```

1. 一般驗證機制設定
2. 建立一個包含 `@Order` 註解的 `WebSecurityConfigurerAdapter` 實例，以指明哪一個 `WebSecurityConfigurerAdapter` 要先被考慮
3. `http.antMatcher` 描述了這個 `HttpSecurity` 只會適用於開頭為 `/api/` 的 URL
4. 建立另一個 `WebSecurityConfigurerAdapter` 實例，如果 URL 開頭不是 `/api/` 則此設定會被使用。因為此設定的 `@Order` 值在 1 之後 (沒有 `@Order` 則預設為最後)，所以此設定會在 `ApiWebSecurityConfigurationAdapter` 之後才會被考慮