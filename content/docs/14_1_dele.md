---
title: "14.1. `DelegatingFilterProxy`"
type: docs
---

# 14.1. `DelegatingFilterProxy`

當使用 servlet filter 的時候，你必須在 `web.xml` 裡面做宣告，否則 filter 就會被 servlet 容器忽略。

在 Spring Security 裡面，filter 類別同時也是定義在 application context 的 Spring bean，因此能夠利用 Spring 的 DI 功能及生命週期介面的優勢，Spring 的 `DelegatingFilterProxy` 可以將 `web.xml` 連結到 application context。

當使用 `DelegatingFilterProxy` 的時候，你會在 `web.xml` 中看到類似這樣的東西：

```xml
<filter>
    <filter-name>myFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>myFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

注意宣告的 filter 其實是 `DelegatingFilterProxy`，而不是會實作 filter 邏輯的類別。

`DelegatingFilterProxy` 會透過一個從 Spring application context 取得的 bean 來代理 `Filter` 的方法，使得這個 bean 能利用 Spring 對 web application context lifecycle 的支援以及設定的彈性。

這個 bean 必須實作 `javax.servlet.Filter` 而且名稱必須與 `filter-name` 元素宣告的一樣。更多細節請參考 [JavaDoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/DelegatingFilterProxy.html)。