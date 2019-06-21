---
title: "18.2. 簡單雜湊 Token 方式"
---

# 18.2. 簡單雜湊 Token 方式

這個方式使用雜湊來達到實用的記得我策略。簡單來說，在成功的驗證過後，一個 cookie 會被發送給瀏覽器，這個 cookie 會長得像這樣：

```
base64(username + ":" + expirationTime + ":" +
md5Hex(username + ":" + expirationTime + ":" password + ":" + key))

username:          可被 UserDetailsService 識別
password:          與取得的 UserDetails 相符
expirationTime:    以毫秒為單位，紀錄記得我 token 到期的日期及時間
key:               用來避免記得我 token 被修改的私有 key
```

這樣的記得我 token 只會依照設定的時間存活，前提是使用者名稱、密碼及 key 都不變。

要注意的是，這個機制有個安全問題，就是當記得我 token 被捕獲後，在 token 失效前，任何使用端都可以使用它，這與摘要驗證是同樣的問題。如果一個請求端取得一個 token，它就可以很容易地變更密碼，並立即讓其他記得我 token 全都失效。

所以如果你需要更加強的安全，你應該參考下一段的機制，或者根本不要使用記得我功能。

如果你對於命名空間設定還算熟悉，你可以加上 `<remember-me>` 元素就可以啟用記得我驗證：

```xml
<http>
    ...
    <remember-me key="myAppKey"/>
</http>
```

通常 `UserDetailsService` 會自動被選用，但如果應用程式裡有不止一個 `UserDetailsService`，你需要在 `user-service-re` 性質中註明要使用的是哪一個 `UserDetailsService` bean 的名稱。