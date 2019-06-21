---
title: "5.10. 方法層級安全"
---

# 5.10. 方法層級安全

從 2.0 開始，Spring Security 大幅度增加了對服務層方法安全的支援，提供對 [JSR-250](https://en.wikipedia.org/wiki/JSR_250) 安全註解的支援，以及框架原生的 `@Secure` 註解。

從 3.0 開始，你還可以使用新的 [表達式註解](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#el-access)。

你可以對一個 bean 作安全控制、可以在宣告 bean 時使用 `intercept-methods` 元素，或者也可以使用 AspectJ 形式的 pointcut 來保護整個服務層的多個 bean。

## 5.10.1. `@EnableGlobalMethodSecurity`

我們可以透過在任何有 `@Configuration` 註解的實例加上 `@EnableGlobalMethodSecurity` 以開啟註解式安全控制功能。

例如以下範例可以開啟 Spring Security 的 `@Secured` 註解功能：

```java
@EnableGlobalMethodSecurity(securedEnabled = true)
public class MethodSecurityConfig {
    // ...
}
```

接著在類別或介面中的方法加上註解，就可以根據註解來限制調用方法的權限。

Spring Security 原生的註解支援對方法設定一組屬性，這些屬性會被傳遞給 `AccessDecisionManager` 以實際作出決定。

```java
public interface BankService {

    @Secured("IS_AUTHENTICATED_ANONYMOUSLY")
    public Account readAccount(Long id);

    @Secured("IS_AUTHENTICATED_ANONYMOUSLY")
    public Account[] findAccounts();

    @Secured("ROLE_TELLER")
    public Account post(Account account, double amount);
}
```

若要開啟對 JSR-250 註解的支援：

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig {
    // ...
}
```

然後相等的 Java 程式碼：

```java
public interface BankService {

    @PreAuthorize("isAnonymous()")
    public Account readAccount(Long id);

    @PreAuthorize("isAnonymous()")
    public Account[] findAccounts();

    @PreAuthorize("hasAuthority('ROLE_TELLER')")
    public Account post(Account account, double amount);
}
```

## 5.10.2. `GlobalMethodSecurityConfiguration`

有時候你可能會要執行一些比 `@EnableGlobalMethodSecurit` 允許的更複雜的行動，這種情況下，你可以繼承 `GlobalMethodSecurityConfiguration` 以確保 `@EnableGlobalMethodSecurity` 註解會出現在繼承後的類別上。

舉例來說，如果你想要提供一個自訂的 `MethodSecurityExpressionHandler`，你可以使用以下設定：

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
	@Override
	protected MethodSecurityExpressionHandler createExpressionHandler() {
		// ... 建立並返回自訂的 MethodSecurityExpressionHandler ...
		return expressionHandler;
	}
}
```

想知道可以被 override 的方法的更多資訊，請參考 [`GlobalMethodSecurityConfiguration` 的 Javadoc](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/method/configuration/GlobalMethodSecurityConfiguration.html)。

## 5.10.3. `EnableReactiveMethodSecurity`

沒在用 Webflux，先跳過。