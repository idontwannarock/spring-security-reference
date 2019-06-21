---
title: "14. Security Filter Chain"
type: docs
---

# 14. Security Filter Chain

Spring Security 的 web 部分基礎完全是建立在標準 servlet filter 上，並不使用 servlet 或在內部使用任何基於 servlet 的框架，例如 Spring MVC，所以並沒有跟特定 web 技術有強烈的關聯。

它處理 `HttpServletRequest` 跟 `HttpServletResponse`，並不管請求是從瀏覽器、web service 客戶端、`HttpInvoker` 或 AJAX 應用而來。

Spring Security 內部會維護一個 filter chain，其中的每個 filter 都有特定的責任，而且會依據需要哪一些服務而決定適用哪些 filter。

filter 的順序很重要，因為它們之間是有依賴關係的。

如果你曾經使用命名空間的方式做設定，那 Spring Security 就已經自動幫你設定好 filters，你不用特別定義任何 Spring bean。但當你要使用某些命名空間方式不支援的功能，或要使用自訂的類別時，你可能就想要完全掌控 security filter chain。