---
title: "25.2. 觸發前處理"
---

# 25.2. 觸發前處理

我們在 [9.5.2. Secure Object 及 `AbstractSecurityInterceptor`]({{< relref "/docs/9_5_acce_cont_auth.md#9-5-2-secure-object-及-abstractsecurityinterceptor" >}}) 也看到，Spring Security 提供許多攔截器來控制 secure object 的存取，例如方法觸發或網路請求。

一個是否允許觸發的觸發前決策是由 `AccessDecisionManager` 來做。

## 25.2.1. `AccessDecisionManager`

`AccessDecisionManager` 是由 `AbstractSecurityInterceptor` 來呼叫，並且負責做最終存取控制決策。

`AccessDecisionManager` 介面包含三個方法：

```java
void decide(Authentication authentication, Object secureObject, Collection<ConfigAttribute> attrs) throws AccessDeniedException;

boolean supports(ConfigAttribute attribute);

boolean supports(Class clazz);
```

`AccessDecisionManager` 所需用來做出授權決策的相關資訊都會被傳入 `decide()` 方法，特別是將 secure `Object` 傳入，以對真正傳入 secure object 觸發過程的參數進行檢驗。

舉例來說，假設 secure object 是一個 `MethodInvocation`，它就可以很簡單的查詢 `MethodInvocation` 是否有任何的 `Customer` 參數，並在 `AccessDecisionManger` 實作某種安全邏輯來確保使用端被允許對該 customer 進行操作，並且實作應該要在存取被拒的時候拋出 `AccessDeniedException`。

`supports(ConfigAttribute)` 方法會在 `AbstractSecurityInterceptor` 初始時被觸發，以決定該 `AccessDecisionManager` 是否可以處理傳入的 `ConfigAttribute`；而 `supports(Class)` 方法則會被某個安全攔截器的實作觸發，以確保該 `AccessDecisionManager` 支援該攔截器會攔截的 secure object 類型

## 25.2.2. 投票機制的 `AccessDecisionManager` 實作

