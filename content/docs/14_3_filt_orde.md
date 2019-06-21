---
title: "14.3. Filter 排序"
---

# 14.3. Filter 排序

在 filter chain 定義好的順序是非常重要的，不論你實際使用哪些 filter，順序都應該如下：

- `ChannelProcessingFilter`：因為此 filter 負責轉導到不同的協定 (protocol)
- `SecurityContextPersistenceFilter`：此 filter 需排在此，如此才能在請求的一開始就將 `SecurityContext` 設定到 `SecurityContextHolder` 當中，而且 `SecurityContext` 的任何變動才能在請求結束 (準備好給下一次請求使用) 時複製到 `HttpSession` 當中
- `ConcurrentSessionFilter`：此 filter 會使用 `SecurityContextHolder` 的功能，而且會更新 `SessionRegistry` 以反應請求端 (principal) 持續進行的請求
- 驗證處理機制：`UsernamePasswordAuthenticationFilter`、`CasAuthenticationFilter`、`BasicAuthenticationFilter` 等等，如此 `SecurityContextHolder` 才能將通過驗證的 `Authentication` 請求 token 包含在內
- `SecurityContextHolderAwareRequestFilter`：如果你要使用這個 filter 來將 Spring Security 的 aware `HttpServletRequestWrapper` 安裝到你的 servlet 容器中
    - Spring Security 的 aware 其實是繼承 `javax.servlet.http.HttpServletRequestWrapper` 的 [`org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestWrapper`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/servletapi/SecurityContextHolderAwareRequestWrapper.html)
- `JaasApiIntegrationFilter`：如果 `SecurityContextHolder` 當中有一個 `JaasAuthenticationToken`，此 filter 會將 `FilterChain` 當成 `JaasAuthenticationToken` 裡面的 `Subject` 處理
- `RememberMeAuthenticationFilter`：當沒有更前面的執行的驗證機制對 `SecurityContextHolder` 做更新，而且該請求提供了一個允許記得我功能啟動的 cookie，則此 filter 會將一個合適的 `Authentication` 物件放到 `SecurityContextHolder` 中
- `AnonymousAuthenticationFilter`：當沒有更前面的執行的驗證機制對 `SecurityContextHolder` 做更新，一個匿名的 `Authentication` 物件會被放到 `SecurityContextHolder` 中
- `ExceptionTranslationFilter`：此 filter 用來接住任何 Spring Security 的 exceptions，然後回傳一個 HTTP 錯誤回應，或啟動一個合適的 `AuthenticationEntryPoint`
- `FilterSecurityInterceptor`：保護 URIs 並在訪問被拒絕的時候拋出異常