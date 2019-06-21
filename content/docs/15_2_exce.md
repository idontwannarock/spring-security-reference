---
title: "15.2. `ExceptionTranslationFilter`"
---

# 15.2. `ExceptionTranslationFilter`

在 security filter chain 當中，`ExceptionTranslationFilter` 是位於 `FilterSecurityInterceptor` 的上一層，它並不會實際的做任何強制性的安全動作，而是處理安全相關的 interceptor 所拋出的異常並提供適合的 HTTP 回應。

```xml
<bean id="exceptionTranslationFilter" class="org.springframework.security.web.access.ExceptionTranslationFilter">
    <property name="authenticationEntryPoint" ref="authenticationEntryPoint"/>
    <property name="accessDeniedHandler" ref="accessDeniedHandler"/>
</bean>

<bean id="authenticationEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
    <property name="loginFormUrl" value="/login.jsp"/>
</bean>

<bean id="accessDeniedHandler" class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
    <property name="errorPage" value="/accessDenied.htm"/>
</bean>
```

## 15.2.1. `AuthenticationEntryPoint`

當使用者針對某個受保護的 HTTP 資源做出請求，但還未通過驗證時，`AuthenticationEntryPoint` 就會被呼叫，然後合適的 `AuthenticationException` 或 `AccessDeniedException` 會被 interceptor 拋出，並觸發該進入點的 `commence` 方法，如此就可以提供適合的回應給使用者以開始驗證流程。在此我們使用的是 `LoginUrlAuthenticationEntryPoint`，它會將請求轉導到一個不同的 URL，通常是一個登入頁面。

實際的實作則要看你想要在你的應用程式中使用怎樣的驗證機制。

## 15.2.2. `AccessDeniedHandle`

如果一個使用者已經驗證過，並且想要訪問一個受保護的資源，會發生什麼事呢？一般來說這應該不會發生，因為應用程式的工作流程應該限制在使用者擁有訪問權限的那些行動。

舉例來說，一個沒有管理者權限的使用者應該看不到可以連到管理頁面的 HTML 連結，當然你不能只依賴隱藏連結的方式，畢竟總是有可能使用者會直接輸入 URL 來試圖繞過這種限制，或可能會修改一個 RESTful URL 的參數值。

你的應用程式應該想辦法應對這些情況，不然肯定會很不安全。

你通常會針對基本的 URL 使用簡單的網路層安全限制，並在服務層介面使用方法安全基礎的限制來真正決定哪些是可允許的資源。

如果一個使用者已通過驗證但又拋出一個 `AccessDeniedException`，就代表試圖進行一個擁有的權限不足以執行的動作。這種情況，`ExceptionTranslationFilter` 會採取第二種策略，`AccessDeniedHandler`。預設會使用 `AccessDeniedHandlerImpl` 來回傳一個 403 (Forbidden) 回應給使用端，但你也可以特別設定一個實例 (如同前面的範例) 來將請求轉導到一個錯誤頁面 URL，這可以是一個簡單的「拒絕訪問」頁面，例如一個 JSP，或可以是一個更複雜的處理器，例如一個 MVC 的 controller。當然你也可以有自己實作的介面。

當然也可以提供一個使用命名空間自訂的 `AccessDeniedHandler` 給你的應用程式，細節可以參考有關命名空間的附錄。

## 15.2.3. `SavedRequest` 及 `RequestCache` 介面

另一個 `ExceptionTranslationFilter` 負責的工作就是在觸發 `AuthenticationEntryPoint`，使得使用者通過驗證後可以恢復原本的請求，參考 [9.4. 網路應用程式的權限驗證]({{< relref "/docs/9_4_auth_in_web_app.md" >}})。典型的例子就是使用者透過表單登入後，被預設的 `SavedRequestAwareAuthenticationSuccessHandler` (參考 [15.4.1. 應用程式驗證成功或失敗的工作流程]({{< relref "/docs/15_4_user.md#15-4-1-應用程式驗證成功或失敗的工作流程" >}})) 導向原始的 URL。

`RequestCache` 封裝了保存及取得 `HttpServletRequest` 實例所需的功能，而預設會使用 `HttpSessionRequestCache` 這個實作來將請求存放到 `HttpSession`；而當使用者要被轉導到原始的 URL 時，`RequestCacheFilter` 則負責實際將保存的請求從緩存中恢復。

正常情況下，你不需要調整任何功能，但因為處理保存的請求採取的是「最佳嘗試」策略，可能還是有些情況超出預設設定可以處理的範圍，所以從 Spring Security 3.0 開始，這些介面的就可以用來自由的做安插。