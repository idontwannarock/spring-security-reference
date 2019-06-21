---
title: "I. 前言"
type: docs
---

# I. 前言

Spring Security 為 Java EE 企業級應用程式提供了一個全面的安全方案，在探索本手冊的過程中，你會發現我們試圖提供一個實用且容許高度客製化的安全系統。

安全是一個需要持續追求的目標，需要一個全面的系統性方法。我們鼓勵將安全分層，而每一層都試圖在其範圍內盡量達到安全，這樣連續性的安全層級會更有保障。而且每層的安全聯繫越緊密，你的應用程式就會更穩固與安全。

在底層，你需要處理如傳輸安全及系統識別等問題，以減輕 [MITM 中間人攻擊](https://zh.wikipedia.org/zh-tw/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)；接著你通常會利用防火牆，可能搭配 VPN 或 [IPsec](https://zh.wikipedia.org/zh-tw/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB) 以確保只有通過授權的系統可以嘗試連接。在企業環境，你可能會佈署一個 [DMZ](https://zh.wikipedia.org/wiki/DMZ) 將公開的伺服器從後端資料庫和應用程式伺服器中分離出來。

作業系統也是個重要的腳色，處理像是以無特殊權限使用者身分執行程序，以及最大化檔案系統安全等問題。作業系統也常與其隨附的防火牆一起做設定。希望在這過程當中你還記得要阻擋 [DOS](https://zh.wikipedia.org/wiki/%E9%98%BB%E6%96%B7%E6%9C%8D%E5%8B%99%E6%94%BB%E6%93%8A) 跟 [暴力破解](https://zh.wikipedia.org/wiki/%E6%9A%B4%E5%8A%9B%E7%A0%B4%E8%A7%A3%E6%B3%95) 的攻擊。在監看及回應攻擊方面，[IDS](https://zh.wikipedia.org/wiki/%E5%85%A5%E4%BE%B5%E6%A3%80%E6%B5%8B%E7%B3%BB%E7%BB%9F) 會特別有用，這種系統可以提供如即時阻絕入侵的 TCP/IP 位址等保護的手段。

進入到較高的層級，你會希望對 JVM 做設定以盡量減少可以通過許可的 Java 類型，然後在你的應用程式加上特定領域的安全設定，Spring Security 可以讓你對特定領域做安全設定時更容易。

當然，你必須適當的處理上面提到每個層級的安全，並把每層會碰到的管理層面因素考慮進去，例如 security bulletin monitoring、patching、personnel vetting、審計、change control、engineering management systems、資料備份、災難復原、performance benchmarking、負載監控、集中化日誌、事件回應程序等等。

而即使有 Spring Security 在企業級應用程式安全層級的幫助，你也會發現不同商業領域的問題就有不同的需求。例如銀行應用程式的需求就跟電商的需求不同，電商應用系統也跟企業銷售自動化工具的需求不同。這些不同的需求也讓應用程式的安全有趣(?)、有挑戰性也更有成就感。

一開始請完整閱讀 [1. 準備開始]({{< relref "/docs/1_gett_star.md" >}}) 這個章節，當中會介紹框架，並使用命名空間作設定，讓你可以快速地開始並運作；如果想要更了解 Spring Security 如何運作以及可能需要用到的某些類別，你可以接著參考 [II. 架構及實作]({{< relref "/docs/ii_arch_and_impl.md" >}})；其餘的部分是以比較傳統的方式撰寫，以供需要的時候依需求查看。

但我們還是建議盡量閱讀有關應用程式安全的議題，畢竟 Spring Security 不是所有安全問題的萬靈丹，重要的還是在開始設計應用程式的時候就留意安全性，改造重構並不是個好主意。在建構應用程式的時候，你就應該考慮到淺在的漏洞，例如 [XSS](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)、[CSRF](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0) 及 [連線劫持](https://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D%E5%8A%AB%E6%8C%81) 等。

[OWASP 網站](http://www.owasp.org/) 列出目前前十大網路應用程式的漏洞，並提供許多有用的參考資訊。

希望這個手冊對你有幫助，也歡迎回饋跟建議。

最後，歡迎來到 Spring Security 社群。