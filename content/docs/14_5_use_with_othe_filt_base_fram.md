---
title: "14.5. 與其他 filter 基礎的框架一起使用"
---

# 14.5. 與其他 filter 基礎的框架一起使用

如果你有使用其他也是以 filter 為基礎的框架，那你必須確保 Spring Security 的 filter 最優先，這樣 `SecurityContextHolder` 才能在其他 filter 使用前及時取得資料。

舉例來說，你可能會使用 [SiteMesh](https://zh.wikipedia.org/zh-tw/SiteMesh) 來裝飾你的網頁，或者 [Wicket](https://zh.wikipedia.org/zh-tw/Wicket) 網路框架也是使用 filter 來處理請求。