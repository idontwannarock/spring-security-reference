---
title: "10.1. `AuthenticationManager`、`ProviderManager` 及 `AuthenticationProvider`"
---

# 10.1. `AuthenticationManager`、`ProviderManager` 及 `AuthenticationProvider`

`AuthenticationManager` 是一個介面，而預設的實作就是 `ProviderManager`，而 `ProviderManager` 還會將驗證請求代理給一連串設定好的 `AuthenticationProvider` 去操作，而 `AuthenticationProvider` 會被依序詢問是否能執行驗證，而每個 provider 要馬丟例外，要馬就回傳裝著使用者 context 資訊的 `Authentication` 物件。

常見的 provider 驗證方式就是取得使用端對應的資料並裝進 `UserDetails`，再比對 `UserDetails` 內的密碼及使用者輸入的密碼，這也是 `DaoAuthenticationProvider` 採取的方式。

若驗證通過，則用這個裝著使用端資料的 `UserDetails`，特別是他持有的 `GrantedAuthority`，去構建 `Authentication`，並存放在 `SecurityContext`。

如果使用命名空間標籤 `<authentication-manager>` 做設定，則一個 `ProviderManager` 的實例會在內部被建立並維護，接著再將 provider 透過 `<authentication-provider>` 加入。但如果使用這種方式，則不應該在應用程式 context 中宣告 `ProviderManager` bean。

若不要使用命名空間的方式宣告，則可能會像這樣：

```xml
<bean id="authenticationManager" class="org.springframework.security.authentication.ProviderManager">
  <constructor-arg>
    <list>
      <ref local="daoAuthenticationProvider"/>
      <ref local="anonymousAuthenticationProvider"/>
      <ref local="ldapAuthenticationProvider"/>
    </list>
  </constructor-arg>
</bean>
```

上面這個例子表示我們使用三種 provider，並且會依序試圖執行驗證，或跳過驗證只回傳 null。

如果每個 provider 都回傳 null，`ProviderManager` 就會拋出 `ProviderNotFoundException`。

在驗證機制 authentication mechanism 中，例如 web form-login processing filter，會被注入 `ProviderManager` 的參考，並且會呼叫它處理驗證請求。

## 10.1.1. 成功驗證後清除憑證 Credentials

從 3.1 開始，Spring Security 就預設 `ProviderManager` 會在成功驗證後嘗試清除被返回的 `Authentication` 內的敏感憑證資訊，以避免像是密碼這種資訊被系統不合理的持有過久。

但這在你用緩存 cache 保留使用者物件的時候可能會造成問題。

例如為了提升無狀態紀錄應用程式的效能，在 `Authentication` 保留了在緩存裡一個物件的參考，例如是一個 `UserDetails` 實例，但一旦因為這個機制將憑證移除後，就沒辦法再對緩存裡的資訊作驗證了，所以如果你有使用緩存，可能需要額外考慮到這個問題。

當然，有一個明顯的解決辦法就是預先把物件複製一份，不論是在緩存的實作中，或是會在建立返回的 `Authentication` 物件的 `AuthenticationProvider` 中。

還有一個辦法，就是停用 `ProviderManager` 的 `eraseCredentialsAfterAuthentication` 屬性，參考 [ProviderManager API](https://docs.spring.io/spring-security/site/docs/5.0.7.RELEASE/api/org/springframework/security/authentication/ProviderManager.html)。

## 10.1.2 `DaoAuthenticationProvider`

最簡單的 `AuthenticationProvider` 實作就是 `DaoAuthenticationProvider`。

`DaoAuthenticationProvider` 將 `UserDetailsService` 當作 DAO 使用來取得 username、password 及 `GrantedAuthority`，它驗證的機制只會比對使用端提交給 `UsernamePasswordAuthenticationToken` 的密碼及 `UserDetailsService` 取得的密碼。

要設定這個 provider 也很簡單，例如：

```xml
<bean id="daoAuthenticationProvider" class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
  <property name="userDetailsService" ref="inMemoryDaoImpl"/>
  <property name="passwordEncoder" ref="passwordEncoder"/>
</bean>
```

`PasswordEncoder` 是選用的，它會對 `UserDetailsService` 取得並放到 `UserDetails` 的密碼做編碼及解碼。