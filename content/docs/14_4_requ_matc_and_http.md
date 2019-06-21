---
title: "14.4. 請求匹配與 `HttpFirewall`"
type: docs
---

# 14.4. 請求匹配與 `HttpFirewall`

Spring Security 有好幾個區塊會將請求與你定義的樣式做比對，以決定如何處理請求，尤其當 `FilterChainProxy` 決定哪一條 filter 鍊要處理請求，以及當 `FilterSecurityInterceptor` 決定哪些安全限制要適用到請求上的時候會發生，所以了解機制跟比對樣式時會使用什麼 URL 會被使用是很重要的。

Servlet 規格在 `HttpSevletRequest` 裡面規範了一些可以透過 getter 取得的屬性，包括 `contextPath`、`servletPath`、`pathInfo` 及 `queryString` 可供我們比對。Spring Security 保護只針對應用程式內的路徑，所以 `contextPath` 可以忽略，但 Servlet 並沒有明確規範針對一個特定請求的 URI，`servletPath` 跟 `pathInfo` 會包含什麼值。

舉例來說，依照 [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) 的規格，URL 的每個區塊都可以包含參數，而 Servlet 規格並沒有明確指出這部分應該被歸類到 `servletPath` 或 `pathInfo`，而且不同的 servlet 容器還會有不同的實作。所以當應用程式被部署到一個不會卸除路徑參數的容器中，攻擊者就可以透過在路徑上做文章，使得比對樣式時成功或非預期的失敗，造成危險。

URL 也可能有其他狀況，例如包含路徑遍歷符號，例如 `/../`，或多個斜線 `//`，因而造成樣式比對失敗。有些容器會對此作處理，但也有一些不會。為了處理這樣的問題，`FilterChainProxy` 使用 `HttpFirewall` 的策略去檢查並包裝請求。未處理的請求預設就會自動被拒絕，而為了比對，路徑參數及多重斜線會被移除，所以運用 `FilterChainProxy` 來處理 security filter chain 是非常關鍵的。

注意容器會將 `servletPath` 跟 `pathInfo` 的值解碼，所以你的應用程式不應該有任何包含分號的合格路徑，因為為了比對，這部分會被移除。

如上所述，預設的比對方式是使用 ANT 風格的路徑，這可能也是多數使用者最好的選擇。這個比對方式由 `AntPathRequestMatcher` 實作，使用了 Spring 的 `AntPathMatcher` 來對忽略 `queryString` 後串聯起來的 `servletPath` 及 `pathInfo` 進行不分大小寫的樣式比對。

如果因為某些理由，你必須要有更強力的比對方式，你可以使用正則表達式，實作在 `RegexRequestMatcher` 中，更多資訊請參考 [JavaDoc](https://docs.spring.io/spring-security/site/docs/5.0.x-SNAPSHOT/api/org/springframework/security/web/util/matcher/RequestHeaderRequestMatcher.html)。

習慣上我們會建議你在服務層使用方法安全來對你的應用程式做訪問控制，而且不要完全依賴在網路應用程式層訂定的安全限制，因為 URL 會改變，而且很難考慮到應用程式所有可能支援的路徑，以及請求會被如何處理。你應該試著只使用幾個簡單易懂的 ANT 風格路徑，並永遠採取「預設拒絕」的方式，也就是把萬用字元 (例如 `/`) 定義在最後並拒絕訪問。

定義在服務層的安全更為穩固而且難以繞過，所以你應該總是利用 Spring Security 提供的方法安全選項的好處。

`HttpFirewall` 也透過拒絕在 HTTP 反應標頭出現換行符號來避免 [HTTP 反應標頭拆分漏洞](https://www.owasp.org/index.php/HTTP_Response_Splitting) 的問題。

預設 `StrictHttpFirewall` 會被使用，這個實作會拒絕惡意請求。如果這個實作的規則對你來說太嚴格，那你也可以自訂什麼樣的請求要被拒絕，但你必須要明白這樣做可能會讓你的應用程式面對風險。

舉例來說，如果你想要使用 Spring MVC 的矩陣參數，你可以在 XML 中做以下設定：

```xml
<b:bean id="httpFirewall"
      class="org.springframework.security.web.firewall.StrictHttpFirewall"
      p:allowSemicolon="true"/>

<http-firewall ref="httpFirewall"/>
```

以暴露 `StrictHttpFirewall` 的方式，用 Java 設定也可以達到同樣的效果：

```java
@Bean
public StrictHttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowSemicolon(true);
    return firewall;
}
```