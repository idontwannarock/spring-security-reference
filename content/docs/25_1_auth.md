---
title: "25.1. 權限"
type: docs
---

# 25.1. 權限

如同我們在 [9.2.3. `GrantedAuthority`]({{< relref "/docs/9_2_core_comp.md#9-2-3-grantedauthority" >}}) 當中看到的，所有的 `Authentication` 實作都保存了一個 `GrantedAuthority` 物件的清單，這些代表使用端被賦予的權限。`GrantedAuthority` 是由 `AuthenticationManager` 注入 `Authentication` 物件，並且會在後續由 `AccessDecisionManager` 在做出授權決策時讀取。

`GrantedAuthority` 是一個只有一個方法的介面：

```java
String getAuthority();
```

這個方法讓 `AccessDecisionManager` 可以取得代表該 `GrantedAuthority` 的精確的 `String`。透過回傳 `String`，`GrantedAuthority` 就可以被大部分的 `AccessDecisionManager` 「讀取」。如果 `GrantedAuthority` 無法精確地用 `String` 來代表，那這個 `GrantedAuthority` 就會被視為是「複雜的」，並且 `getAuthority()` 方法必須回傳 `null`。

舉例來說，一個「複雜的」`GrantedAuthority` 實作可能會保存一份動作清單以及對應不同客戶帳號的不同權限門檻。要用一個 `String` 來代表這樣的複雜 `GrantedAuthority` 相當的困難，所以 `getAuthority()` 方法應該要回傳 `null`，這會指示所有 `AccessDecisionManager` 必須要針對這個 `GrantedAuthority` 實作有特別的支援以理解其內容。

Spring Security 有一個 `GrantedAuthority` 的實作，`SimpleGrantedAuthority`，它允許任何使用者自定義的 `String` 轉換為一個 `GrantedAuthority`。所有包含在架構內的 `AuthenticationProvider` 都使用 `SimpleGrantedAuthority` 來填入 `Authentication` 物件。