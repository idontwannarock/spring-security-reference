---
title: "V. 授權管理"
---

# V. 授權管理

Spring Security 進階的授權管理功能，是其受到歡迎最引人注目的原因之一。不管你選擇怎麼做驗證，不論是使用 Spring Security 提供的機制跟提供者，或是與一個容器或其他非 Spring Security 的驗證方整合，你都能在你的應用程式中穩定簡單的使用授權管理。

在這個段落，我們會探討不同的 `AbstractSecurityInterceptor` 實作，然後會進一步探討如何用 domain 存取控制表來更進一步微調授權管理。