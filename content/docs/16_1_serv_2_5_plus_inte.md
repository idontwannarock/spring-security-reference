---
title: "16.1. Servlet 2.5+ 整合"
type: docs
---

# 16.1. Servlet 2.5+ 整合

## 16.1.1. `HttpServletRequest.getRemoteUser()`

[`HttpServletRequest.getRemoteUser()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getRemoteUser()) 方法會回傳 `SecurityContextHolder.getContext().getAuthentication().getName()` 的結果，一般來說就是目前使用者的 username。

這功能在你想要在應用程式中展示目前使用者的 username 時很有用，另外檢查這個值是否為 null 也可以被用來辨識一個使用者是否有經過驗證或是一個匿名使用者。當要決定某個 UI 元件 (例如應該只有已驗證使用者可以看得到登出連結) 是否要顯示時，知道使用者是否經過驗證就很有用了。

## 16.1.2. `HttpServletRequest.getUserPrincipal()`

[`HttpServletRequest.getUserPrincipal()`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#getUserPrincipal()) 方法則會回傳 `SecurityContextHolder.getContext().getAuthentication()` 的結果。這表示這個物件是一個 `Authentication`，而當使用 username 和 password 為基礎的驗證方式時，它通常是 `UsernamePasswordAuthenticationToken` 的一個實例。

當你需要知道更多關於使用者資訊時，這就很有用了。舉例來說，你可能需要自訂一個可以回傳包含使用者姓跟名的客製 `UserDetails` 的 `UserDetailsService`，你可以這樣來取得資訊：

```java
Authentication auth = httpServletRequest.getUserPrincipal();
// 假設已整合的自訂 UserDetails 叫做 MyCustomUserDetails
// 預設它通常是一個 UserDetails 的實例
MyCustomUserDetails userDetails = (MyCustomUserDetails) auth.getPrincipal();
String firstName = userDetails.getFirstName();
String lastName = userDetails.getLastName();
```

> 值得注意的是在整個應用程式中作這麼多邏輯操作通常是個壞習慣，我們應該將將它集中以盡量降低 Spring Security 跟 Servlet API 的耦合

## 16.1.3. `HttpServletRequest.isUserInRole(String)`

[`HttpServletRequest.isUserInRole(String)`](https://docs.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html#isUserInRole(java.lang.String)) 方法會判斷 `SecurityContextHolder.getContext().getAuthentication().getAuthorities()` 是否包含了被傳入 `isUserInRole(String)` 腳色的 `GrantedAuthority`。

通常使用者不應該將 `ROLE_` 這個前綴字一併傳入這個方法，因為它會被自動加上去。例如如果你想判斷目前使用者是否有 `ROLE_ADMIN` 的權限，你可以這樣做：

```java
boolean isAdmin = httpServletRequest.isUserInRole("ADMIN");
```

這個功能在當你要決定某個 UI 元件是否要被顯示時可能會有用，譬如只有管理員使用者可以看到管理員連結。