---
title: "15.4. `UsernamePasswordAuthenticationFilter`"
type: docs
---

# 15.4. `UsernamePasswordAuthenticationFilter`

我們已經看了三個 Spring Security 網路設定中一直會出現的三個主要的 filter，這三個 filter 是由命名空間 `<http>` 元素自動產生，而且無法被其他替代方案抽換。

現在唯一還沒提到的是一個真正的驗證機制，一個真正讓使用者驗證的東西。

這個 filter 是最廣泛被使用的驗證 filter，也是最常被客製的，它也提供給命名空間 `<form-login>` 元素使用的實作。

它有三個需要作設定的階段：

- 設定一個含有登入頁面 URL 的 `LoginUrlAuthenticationEntryPoint`，如同我們前面所做的那樣，並把它設定到 `ExceptionTranslationFilter`
- 實作一個登入頁面，不論是 JSP 或 MVC 的 controller
- 在 application context 中設定一個 `UsernamePasswordAuthenticationFilter` 的實例
- 將此 filter 的 bean 加入你的 filter chain proxy，但請務必注意順序

登入的頁面只會包含 `username` 及 `password` 的輸入框，並用 POST 請求到被 filter 監控的 URL，預設為 `/login`。

基本的 filter 設定會長得像這樣：

```xml
<bean id="authenticationFilter" class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
    <property name="authenticationManager" ref="authenticationManager"/>
</bean>
```

## 15.4.1. 應用程式驗證成功或失敗的工作流程

這個 filter 會呼叫設定好的 `AuthenticationManager` 來處理每個驗證請求。而不論是驗證成功或失敗的目的則是對應的由 `AuthenticationSuccessHandler` 或 `AuthenticationFailureHandler` 介面來控制。這個 filter 裡面有屬性可以讓你完全的自訂這些行為。

有些標準的實作例如 `SimpleUrlAuthenticationSuccessHandler`、`SavedRequestAwareAuthenticationSuccessHandler`、`SimpleUrlAuthenticationFailureHandler`、`ExceptionMappingAuthenticationFailureHandler` 及 `DelegatingAuthenticationFailureHandler`。參考這些類別的 JavaDoc 連同 `AbstractAuthenticationProcessingFilter` 一起看會讓你對它們如何作業及有哪些功能有一個概念。

一旦驗證成功，產出的 `Authentication` 物件會被放到 `SecurityContextHolder` 當中，接著 `AuthenticationSuccessHandler` 會被呼叫來決定是要轉導還是將使用者轉向合適的目的。預設會使用 `SavedRequestAwareAuthenticationSuccessHandler`，代表使用者會被轉到當初被要求登入前請求的原始目的。

> `ExceptionTranslationFilter` 會緩存使用者最原始的請求。當使用者作驗證時，請求處理器會使用此緩存的請求來取得原始的 URL 並轉導到該處，然後這個原始請求就會被重建並被替換

如果驗證失敗，則設定好的 `AuthenticationFailureHandler` 會被觸發。