---
title: "17.2. `DigestAuthenticationFilter`"
type: docs
---

# 17.2. `DigestAuthenticationFilter`

`DigestAuthenticationFilter` 可以處理 HTTP 標頭內的摘要驗證憑證。

摘要驗證嘗試解決基本驗證的許多弱點，特別是為了避免憑證不會被以明文傳遞。

許多使用者端支援摘要驗證，包括 Mozilla Firefox 及 IE。摘要驗證的標準處理程序定義在 RFC 2617，並且更新了 RFC 2069 這個較早期的版本，多數的使用者端都實作了 RFC 2617。而 `DigestAuthenticationFilter` 兼容 RFC 2617 規範的「auth」[qop](https://tools.ietf.org/html/rfc2617#section-3.2.1) 保護質量指示語，也向下支援 RFC 2069。

如果你需要使用無加密 HTTP 方式，例如沒有 TLS/HTTPS，而且希望加強驗證過程的安全，那摘要驗證就是一個很有吸引力的選項。而在 RFC 2518 段落 17.1 當中，摘要驗證也的確是 [WebDAV 協議](https://zh.wikipedia.org/zh-tw/%E5%9F%BA%E4%BA%8EWeb%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E7%BC%96%E5%86%99%E5%92%8C%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6) 必備的。

> 你不應該在當代的應用程式中使用摘要，因為它已經被認為是不安全的，最明顯的問題就是它會將密碼以純文字、加密或 MD5 格式保存，這些格式都被認為是不安全的。你應該使用單向雜湊密碼，例如 bCript、PBKDF2、SCript 等等。

摘要驗證的核心是 [nonce](https://zh.wikipedia.org/zh-tw/Nonce)，它是個伺服器產生的值。

Spring Security 採取以下格式產生 nonce：

```
base64(expirationTime + ":" + md5Hex(expirationTime + ":" + key))

expirationTime:   以毫秒為單位，紀錄 nonce 失效的日期時間
key:              用來避免 nonce token 被更改的私有 key
```

`DigestAuthenticatonEntryPoint` 有一個屬性用來註明產生 nonce token 的 `key`，還有一個 `nonceValiditySeconds` 屬性註明到期時限，預設是 300，也就是五分鐘。只要 nonce 還是合格的，摘要就會透過串聯不同的字串，如 username、password、nonce、請求 URI、客戶端產生 nonce (客戶端每次請求產生的隨機值)、領域名稱等等，來產生摘要，然後再進行 MD5 雜湊。

伺服器端跟使用者端都會進行摘要運算，所以只要包含的值不同，例如密碼，就會產生不同的雜湊值。

在 Spring Security 的實作中，如果伺服器端產的 nonce 剛剛過期，但摘要還是合格的，`DigestAuthenticationEntryPoint` 會傳送一個 `stale=true` 的標頭，這會告訴使用者端不用打擾使用者，因為密碼及使用者名整等等都還是正確的，只需要重新以新的 nonce 嘗試一次就可以了。

對於 `DigestAuthenticationEntryPoint` 的 `nonceValiditySeconds` 來說合適的值要依賴你的應用程式，極度安全的應用程式應該注意被打斷的驗證標頭，在 nonce 包含的 `expirationTime` 到達之前，可能會被用來假扮成某個使用者，這就是選擇一個合適設定關鍵的原則。不過另一方面來說，對於非常要求安全的應用程式，不在一開始就採用 TLS/HTTPS 也是很不尋常的。

對於更複雜的摘要驗證實作來說，使用者端的問題也很常發生。例如 IE 在同一 session 中連續的請求無法提供 opaque token，所以 Spring Security 會將所有的狀態資訊封裝到 nonce token 當中。

在測試中，Spring Security 與 Mozilla Firefox 及 IE 都可以可靠的工作，如處理 nonce 過期等等。

## 17.2.1. 設定

我們剛講完理論，現在來看看如何來使用它。

要實作 HTTP 摘要驗證，我們必須要將 `DigestAuthenticationFilter` 定義到 filter 鍊當中。application context 也需要定義 `DigestAuthenticationFilter` 以及它需要的合作者：

```xml
<bean id="digestFilter" class="org.springframework.security.web.authentication.www.DigestAuthenticationFilter">
    <property name="userDetailsService" ref="jdbcDaoImpl"/>
    <property name="authenticationEntryPoint" ref="digestEntryPoint"/>
    <property name="userCache" ref="userCache"/>
</bean>

<bean id="digestEntryPoint" class="org.springframework.security.web.authentication.www.DigestAuthenticationEntryPoint">
    <property name="realmName" value="Contacts Realm via Digest Authentication"/>
    <property name="key" value="acegi"/>
    <property name="nonceValiditySeconds" value="10"/>
</bean>
```

因為 `DigestAuthenticationFilter` 能直接取得使用者的明文密碼，所以必須設定 `UserDetailsService`，如果在 DAO 使用編碼過的密碼，摘要驗證是無法運作的。合作的 DAO 以及 `UserCache` 通常是由 `DaoAuthenticationProvider` 所提供。而 `authenticationEntryPoint` 屬性則必須是 `DigestAuthenticationEntryPoint`，這樣 `DigestAuthenticationFilter` 才能取得正確的 `realmName` 以及 `key` 來進行摘要運算。

如同前一章節討論的 `BasicAuthenticationFilter` 一樣，如果驗證成功，產出的 `Authentication` 請求 token 會被放到 `SecurityContextHolder`。如果請求事件成功，或因為 HTTP 標頭沒有包含摘要驗證請求而並無嘗試驗證，filter 鍊還是會照常繼續。只有當驗證失敗而且 `AuthenticationEntryPoint` 被呼叫的時候，filter 鍊才會被打斷。

摘要驗證的 [RFC 請求意見稿](https://zh.wikipedia.org/zh-tw/RFC) 提供了許多能增進安全的額外功能，舉例來說，nonce 可以隨著每次請求而改變。

除此之外，Spring Security 實作的目的是為了要簡化整個實作的複雜度，以及可能因與使用者端不兼容而造成的問題，並且避免保存伺服器端的狀態。

如果你想了解這些功能的細節，你可以參考 [RFC 2617](https://www.ietf.org/rfc/rfc2617.txt)，但起碼目前為止就我們的認知，Spring Security 的實作符合此 RFC 標準的最低要求。