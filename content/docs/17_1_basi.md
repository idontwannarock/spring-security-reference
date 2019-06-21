---
title: "17.1. `BasicAuthenticationFilter`"
---

# 17.1. `BasicAuthenticationFilter`

`BasicAuthenticationFilter` 負責以 HTTP 標頭攜帶的憑證來做基本驗證，它可以用來處理 Spring 遠端協定，如 Hessian 及 Burlap，或普通瀏覽器使用者端，如 Firefox 及 IE 所作的驗證請求。

RFC 1934，段落 11 定義了如何處理標準的 HTTP 基本驗證，`BasicAuthenticationFilter` 符合 RFC 規範。

基本驗證是很吸引人的驗證方式，因為絕大多數的使用者端都有部屬此方式，而且實作上非常簡單，它只是一個在 HTTP 標頭中定義的 Base64 編碼 username:password 字段。

## 17.1.1. 設定

要實作 HTTP 基本驗證，你需要將 `BasicAuthenticationFilter` 加入 filter 鍊。application context 應該包含 `BasicAuthenticationFilter` 以及它需要的合作者：

```xml
<bean id="basicAuthenticationFilter" class="org.springframework.security.web.authentication.www.BasicAuthenticationFilter">
    <property name="authenticationManager" ref="authenticationManager"/>
    <property name="authenticationEntryPoint" ref="authenticationEntryPoint"/>
</bean>

<bean id="authenticationEntryPoint" class="org.springframework.security.web.authentication.www.BasicAuthenticationEntryPoint">
    <property name="realmName" value="Name Of Your Realm"/>
</bean>
```

設定當中的 `AuthenticationManager` 會處理每個驗證請求。

如果驗證失敗，則 `AuthenticationEntryPoint` 會用來重新嘗試驗證程序。你通常會將此 filter 與 `BasicAuthenticationEntryPoint` 結合使用，它會返回一個有著合適標頭的 401 回應以重試 HTTP 基本驗證。

如果驗證成功，產出的 `Authentication` 物件會被放入 `SecurityContextHolder`。

如果驗證是成功的，或者因為 HTTP 標頭並沒有包含支援的驗證請求而沒有嘗試做驗證，filter 鍊還是會照常繼續，所以只有當驗證失敗而且 `AuthenticationEntryPoint` 被呼叫時，此 filter 鍊才會被打斷。