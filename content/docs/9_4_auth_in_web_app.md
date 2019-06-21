---
title: "9.4. 網路應用程式的權限驗證"
---

# 9.4. 網路應用程式的權限驗證

一般典型的 Web 應用程式驗證如下:

1. 訪問主頁，點擊一個連結
2. 產生一個 request 到 server，而 server 判斷你要求的資源被權限控管
3. 因為你目前還沒驗證，server 會回傳一個 response 指出你需要通過驗證，這個 response 可以是一個 HTTP response code，或直接將你導向某個頁面
4. 然後隨著驗證機制的進行，可能瀏覽器會導向某個網頁填寫某些資訊，或瀏覽器透過某種方式取得你的 identity，例如彈出基本的驗證對話框、cookie 等等
5. 瀏覽器再次送出 request 給 server，可能是帶有所填表格資訊的 POST request，或帶有驗證資訊表頭的 GET request
6. 接著 server 會決定得到的憑證 credentials 是否合格，如果是則進行下一步；若否，通常會回到瀏覽器重試一次(第 4 步)
7. 最後，最一開始觸發驗證的請求會重新觸發，若經過驗證的身分所擁有的權限可以取得該資源，則請求成功；若權限不足，則會回傳 HTTP error code 403 "forbidden"

Spring Security 對上述大多數的步驟都有對應的特定類別，主要有關的類別照使用順序依序為 `ExceptionTranslationFilter`、`AuthenticationEntryPoint`，接著是某種 authentication mechanism 驗證機制，此機制會負責呼叫 `AuthenticationManager`。

## 9.4.1. `ExceptionTranslationFilter`

`ExceptionTranslationFilter` 是一個 Spring Security filter，負責偵測任何在 Spring Security 執行過程中丟出的例外，而這些例外通常都是由主要負責資源管理的 `AbstractSecurityInterceptor` 丟出。

所以 `ExceptionTranslationFilter` 要馬就是因為驗證過但權限不足回傳 HTTP error code 403 "forbidden" (上面的第 7 步)；要馬就是還沒驗證，所以啟動 `AuthenticationEntryPoint`。

## 9.4.2. `AuthenticationEntryPoint`

基本上就是負責執行上面的第 3 步，當然有預設的驗證機制，也可以自己調整。

## 9.4.3. 驗證機制 Authentication Mechanism

Spring Security 對於在上述第 6 步負責從使用者端(user agent，通常是瀏覽器)取得憑證並作驗證的過程有個名字，驗證機制 Authentication Mechanism，例如表單登入配合 Basic 驗證。

步驟為一旦從使用者端取得憑證後，一個 `Authentication` _請求_ 物件就被建立並提供給 `AuthenticationManager`。

- 而當驗證機制收到裝著使用者 context 資訊的 `Authentication` 物件後，就會認為目前的請求是合格的。請求被認證為合格後，驗證機制會將 `Authentication` 物件放入 `SecurityContext`，並重新嘗試最初的請求
- 若請求被 `AuthenticationManager` 拒絕，那驗證機制就會要求使用者端再重試一次(上述的第 2 步)

## 9.4.4. 在請求之間保存 `SecurityContext`

在一般網路應用程式，使用者只要登入一次，然後就可以憑 session id 持續通過驗證，而 server 端在 session 沒斷前，會負責保存 principal 資訊。

在 Spring Security 則由 `SecurityContextPersistenceFilter` 負責在請求之間保存 `SecurityContext`，而且會將 `SecurityContext` 以 `HttpSession` 屬性來保存。

它也會負責在每次請求時將 `SecurityContext` 復原給 `SecurityContextHolder`，並在請求結束時清除 `SecurityContextHolder`。

但基本上基於安全理由(而且也沒有什麼其他正當理由)，我們不應該直接操作 `HttpSession` 屬性，不論如何都應該使用 `SecurityContextHolder` 來操作 `SecurityContext`。

注意，有些種類的應用程式，例如 RESTful 的網路服務，並不使用 HTTP session，並且每次請求都會重新驗證；但是仍然建議加入 `SecurityContextPersistenceFilter`，這樣可以確保每次請求結束後 `SecurityContextHolder` 都會被清除。

> 當應用程式在同一個 session 收到 concurrent 請求時，預設會將同一個 `SecurityContext` 分享給每個請求產生的執行緒，即使有用 `ThreadLocal` 也一樣
>
> 所以當你用 `SecurityContextHolder.getContext().setAuthentication`，並因此對 `Authentication` 做了變動，這會影響到所有 concurrent 執行緒的 `Authentication`
>
> 當然你也可以對 `SecurityContextPersistenceFilter` 做設定，使它在每次請求時都建立一個全新的 `SecurityContext` 以避免這個問題