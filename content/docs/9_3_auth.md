---
title: "9.3. 權限驗證"
---

# 9.3. 權限驗證

Spring Security 可以整合很多種不同的驗證環境。

雖然我們建議只使用 Spring Security 來做驗證，而不要整合容器管理的驗證系統，但不論如何它還是支援與自訂的驗證系統做整合。

## 9.3.1. Spring Security 的權限驗證是什麼？

一般普遍的驗證流程例如：

1. 使用者用 username 及 password 登入
2. 系統成功驗證對於該 username 的 password 正確
3. 系統取得有關該使用者的 context 資訊，例如腳色列表等
4. 系統為該使用者建立 security context
5. 使用者繼續執行某些受到控管的行動，資源控管機制會依照 security context 檢查執行該行動的權限

前三步基本組成了權限驗證的程序，而 Spring Security 大概是這樣做的：

1. 取得 username 及 password 並放入 `UsernamePasswordAuthenticationToken` 的實例中
    - `UsernamePasswordAuthenticationToken` 實作了 `Authentication` 介面
2. 該 token 被傳遞給 `AuthenticationManager` 的實例作驗證 validation
3. 若驗證成功，`AuthenticationManager` 會回傳一個裝著該使用者 context 資訊的 `Authentication` 實例
4. Spring Security 容器會接著呼叫 `SecurityContextHolder.getContext().setAuthentication()` 方法，並將剛得到的 `Authentication` 物件當參數傳入，以建立 security context

從這之後，該使用者就會被視為通過驗證，只有 `SecurityContextHolder` 能取得裝著使用者完整 context 資訊的 `Authentication`。

## 9.3.2. 直接設定 `SecurityContextHolder` 內容

Spring Security 並不管你如何將 `Authentication` 裝進 `SecurityContext`，它只要求在 `AbstractSecurityInterceptor` 需要授權某些行動 **之前** 就將 `Authentication` 裝進 `SecurityContext`。

所以很多使用者會客製一個 filter 以從第三方取得使用者資訊來搭建符合 Spring Security 規範的 `Authentication` 物件，再將它放入 `SecurityContext`。這個第三方可以是歷史留存的驗證系統、JNDI 位址等等。

但這種方式需要額外考量一些原本 Spring Security 會自動幫忙處理掉的事情，例如送出 response 之前預先建立好 HTTP session 等等。