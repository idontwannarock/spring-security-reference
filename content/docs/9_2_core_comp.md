---
title: "9.2. 核心組件"
---

# 9.2. 核心組件

## 9.2.1. `SecurityContextHolder`、`SecurityContext` 及 `Authentication` 物件

`SecurityContext` 保存目前系統的 security context 及其細節，由 `SecurityContextHolder` 這個 helper class 提供接觸 `SecurityContext` 的方法。

預設會使用 `ThreadLocal` 來保存 `SecurityContext` 細節，表示同執行緒的方法都可以取得 security context；但其實也可以對 `SecurityContextHolder` 做設定來調整 security context 保存的模式。

Spring Security 在執行緒結束時會自動清除 `ThreadLocal`，所以不用擔心 _ThreadLocal memory leak_ 的問題。

Spring Security 使用 `Authentication` 來紀錄什麼樣的 principal 目前正在跟應用程式互動，而 `Authentication` 也會保存在 `SecurityContext`。

通常不用自己產生 `Authentication`，但可能會在應用程式的各個地方取用 `Authentication`。例如：

```Java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
  String username = ((UserDetails) principal).getUsername();
} else {
  String username = principal.toString();
}
```

其中 `getContext()` 返回的物件是 `SecurityContext` 介面的實例，也就是保存在 `ThreadLocal` 的物件。

而在 Spring Security 多數的驗證機制當中，都是以 `UserDetails` 介面的實例物件作為 principal，所以從 `Authentication.getPrincipal()` 出來的物件通常都可以直接轉型為 `UserDetails` 類別。

但從 `SecurityContextHolder` 取得各種目前使用者的細節不是很符合 DI 的精神，所以如果要在 SpringMVC 的 controller 中使用，建議讓 Spring 提供 `Authentication` 或 `Principal` 物件。

## 9.2.2. `UserDetailsService`

實際上 `Authentication.getPrincipal()` 出來只是個代表 principal 的物件，只是通常可以直接轉型為 `UserDetails`，但其實也可以轉型為自訂的 vo 以利後續邏輯執行。

而通常在應用程式中要取得 `UserDetails` 及其中的資訊，可以透過 `UserDetailsService` 這個介面裡面唯一的方法來取得。

```Java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```

而 `UserDetailsService` 其實只是一種專門用來取得使用者資訊的 DAO，以供其他 Spring Component 使用，他並不會作驗證，實際驗證是由 `AuthenticationManager` 執行。

> 一般來說通常都是實作 `AuthenticationProvider` 來客製驗證機制

如果成功驗證，則會以 `UserDetails` 來構建 `Authentication` 物件並存放在某個 `ThreadLocal` 的 `SecurityContext`，以供隨時透過 `SecurityContextHolder` 取用。

而 `UserDetailsService` 有多種預設的實作去取得 `UserDetails`，例如 `InMemoryDaoImpl` 跟 `JdbcDaoImpl`，或自訂的實作去取得自訂的 principal。

## 9.2.3. `GrantedAuthority`

`Authentication` 除了提供 principal 以外，另一個重點就是用 `getAuthorities()` 方法取得 `GrantedAuthority` 物件的陣列。

> 之所以回傳陣列，是因為每個 principal 可以有多種權限組合

`GrantedAuthority` 物件當然就是 principal 的權限，通常都是某種腳色，譬如 `ROLE_ADMINISTRATOR` 或 `ROLE_HR_SUPERVISOR`。

- 權限大概分三種，web authorization、method authorization 及 domain object authorization，每個腳色針對這三部分都可能有不同的權限組合
- 一般來說不太會針對每個 domain object 作權限控管，否則數量一多 memory 很容易炸開

## 9.2.4. 總結

- `SecurityContextHolder` 是取得 `SecurityContext` 的 helper class
- `SecurityContext` 用來保存 `Authentication` 及可能有針對 request 的 security information
- `Authentication` 會以特定的形式代表某個 principal
- `GrantedAuthority` 代表某個 principal 的權限
- `UserDetails` 是透過 DAO 或其他來源取得的用來建構 `Authentication` 物件必要的資訊
- `UserDetailsService` 預設會透過傳入的 username 字串(或 ID 等)建立 `UserDetails` 物件