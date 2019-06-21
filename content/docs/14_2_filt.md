---
title: "14.2. `FilterChainProxy`"
---

# 14.2. `FilterChainProxy`

Spring Security 的 web 基礎架構應該只被用來代理給一個 `FilterChainProxy` 的實例，security filter 不該被它們自己所使用。

理論上，你可以在 application context 檔案裡面宣告每一個你需要的 Spring Security filter，並在 `web.xml` 內為每個 filter 加上一個對應的 `DelegatingFilterProxy` 進入點，只要確保它們的順序正確。

但這樣寫太笨重，而且當你有一堆 filter 的時候會塞爆 `web.xml` 檔。

`FilterChainProxy` 讓我們可以在 `web.xml` 裡面加入單一的進入點，然後就完全可以在 application context 檔內管理 web security bean。

它們是透過 `DelegatingFilterProxy` 像前面的例子那樣串聯，但將 `filter-name` 設定成 bean 的名稱 `filterChainProxy`，然後在 application context 裡使用同一個名稱宣告 filter chain：

```xml
<bean id="filterChainProxy" class="org.springframework.security.web.FilterChainProxy">
    <constructor-arg>
        <list>
            <sec:filter-chain pattern="/restful/**" filters="
                securityContextPersistenceFilterWithASCFalse,
                basicAuthenticationFilter,
                exceptionTranslationFilter,
                filterSecurityInterceptor" />
            <sec:filter-chain pattern="/**" filters="
                securityContextPersistenceFilterWithASCTrue,
                formLoginFilter,
                exceptionTranslationFilter,
                filterSecurityInterceptor" />
        </list>
    </constructor-arg>
</bean>
```

命名空間元素 `filter-chain` 是用來方便設定應用程式需要的 security filter chain，他會依特定的 URL 樣式去對照一串 filter，這些 filter 是由在 `filters` 元素裡註明名稱的 bean 建立起來的，然後在一個類型是 `SecurityFilterChain` 的 bean 當中將這些 filter 合併起來。

`pattern` 屬性吃 [Ant 風格的路徑](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/util/AntPathMatcher.html)，最特殊的 URI 應該最先出現。

在運行時，`FilterChainProxy` 會找出第一個符合目前請求的 URI 樣式，以及相對應 `filters` 屬性內標註的一連串 filter bean 並適用給該請求，然後 filter 會按照設定的順序被調用，如此你就可以完全掌控對應到特定 URI 的 filter chain。

你可能有注意到在上面的範例中，我們在 filter chain 宣告了兩個 `SecurityContextPersistenceFilter` (`ASC` 是 `SecurityContextPersistenceFilter` 屬性 `allowSessionCreation` 的縮寫)，因為既然 web service 永遠不會在未來的請求顯示 `jsessionid`，那為了這樣的使用端建立 `HttpSession` 就太浪費了。

如果你有一個需要最大擴展性的大型應用程式，我們建議你採取範例中的這種作法；至於小程式，使用單一的 `SecurityContextPersistenceFilter` 加上預設 `allowSessionCreation` 為 `true` 應該就夠了。

有一點要注意，`FilterChainProxy` 不會去調用對應的 filter 裡的標準 filter 生命週期方法，所以我們建議使用 Spring 的 applicatin context 生命週期介面作為替代方式，如同你會對任何 Spring bean 做的一樣。

當我們在看如何使用命名空間來設定網路安全的時候，我們使用了名稱為 springSecurityFilterChain 的 `DelegatingFilterProxy`，你現在應該看的出來這個 `FilterChainProxy` 的名字是由命名空間創造出來的。

## 14.2.1. 繞過 filter chain

你可以使用 `filters = "none"` 屬性代替提供 filter bean 清單，這樣 filter chain 就會完全忽略掉對應的請求樣式，只是要注意任何符合該路徑樣式的請求，就完全不會有權限驗證跟存取控制，因而完全可以自由存取。

如果你想在請求過程中使用 `SecurityContext` 中的內容，則它必須透過 security filter chain 作傳遞，否則 `SecurityContextHolder` 會取不到東西並回傳 null。