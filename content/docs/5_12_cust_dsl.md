---
title: "5.12. 自訂 DSL 領域特定語言"
type: docs
---

# 5.12. 自訂 DSL 領域特定語言

在 Spring Security 中，你也可以提供你自訂的 DSL。

例如你可能會有像以下這樣的東西：

```java
public class MyCustomDsl extends AbstractHttpConfigurer<MyCustomDsl, HttpSecurity> {
	private boolean flag;

	@Override
	public void init(H http) throws Exception {
		// 任何要加上其他 configurer 的方法都必須要在 init 方法中完成
		http.csrf().disable();
	}

	@Override
	public void configure(H http) throws Exception {
		ApplicationContext context = http.getSharedObject(ApplicationContext.class);

        // 此處我們從 ApplicationContext 中取得，但你也可以直接建立一個新的實例
		MyFilter myFilter = context.getBean(MyFilter.class);
		myFilter.setFlag(flag);
		http.addFilterBefore(myFilter, UsernamePasswordAuthenticationFilter.class);
	}

	public MyCustomDsl flag(boolean value) {
		this.flag = value;
		return this;
	}

	public static MyCustomDsl customDsl() {
		return new MyCustomDsl();
	}
}
```

> 其實像是 `HttpSecurity.authorizeRequests()` 這樣的方法也是這樣實作的

然後自訂的 DSL 可以像這樣使用：

```java
@EnableWebSecurity
public class Config extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.apply(customDsl())
				.flag(true)
				.and()
			...;
	}
}
```

其中的程式碼會以下列順序被調用：

1. `Config` 的 `configure()` 方法會先被調用
2. `MyCustomDsl` 的 `init()` 方法會接著被調用
3. `MyCustomDsl` 的 `configure()` 方法會最後被調用

如果你想，也是可以透過使用 `SpringFactories` 的方式讓 `WebSecurityConfiguerAdapter` 預設就加上 `MyCustomDsl`。

例如你可以在 classpath 建立一個 `META-INF/spring.factories` 檔案，內容如下：

```yml
org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer = sample.MyCustomDsl
```

如果使用者要停用預設，可以這樣做：

```java
@EnableWebSecurity
public class Config extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.apply(customDsl()).disable()
			...;
	}
}
```