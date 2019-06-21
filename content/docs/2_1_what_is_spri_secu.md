---
title: "2.1. 什麼是 Spring Security"
type: docs
---

# 2.1. 什麼是 Spring Security

Spring Secuity 好棒棒 blah blah，重點是要熟悉 Spring，特別是 DI 依賴注入的部分。

接著先嗆一頓 Servlet 跟 EJB，再說 Spring Security 好棒棒。

應用程式安全主要分成兩個部分，驗證 (authentication) 以及授權 (authorization | access-control)。驗證就是一個建立 Principal 的過程，Principal 通常代表一個使用者、裝置或其他要對你的應用程式採取動作的系統；授權則是指決定 Principal 在你的應用系統中是否能執行一個動作的過程。而當進行到一個需要授權的階段時，Principal 的身分 (identity) 應該早就透過驗證建立起來。這些是很普遍的概念，並不局限於 Spring Security。

Spring Security 支援多種驗證模型，這些模型大多數是由第三方或相關標準單位所提供，例如 [IETF](https://zh.wikipedia.org/zh-tw/%E4%BA%92%E8%81%94%E7%BD%91%E5%B7%A5%E7%A8%8B%E4%BB%BB%E5%8A%A1%E7%BB%84)。另外，Spring Security 也自訂了一些驗證功能。

Spring Security 目前支援整合以下驗證技術：

- HTTP BASIC authentication headers (an IETF RFC-based standard)
- HTTP Digest authentication headers (an IETF RFC-based standard)
- HTTP X.509 client certificate exchange (an IETF RFC-based standard)
- LDAP (常見的跨平台驗證需求，特別是在大型環境)
- 表單驗證 (簡單的使用者介面需求)
- OpenID authentication
- Authentication based on pre-established request headers (such as Computer Associates Siteminder)
- Jasig Central Authentication Service (otherwise known as CAS, which is a popular open source single sign-on system)
- Transparent authentication context propagation for Remote Method Invocation (RMI) and HttpInvoker (a Spring remoting protocol)
- 自動化「記得我」驗證 (只需要打個勾，就可避免在一段預先設定好的時間內每次都要重新驗證)
- 匿名驗證 (讓每個未經過驗證的呼叫，自動定義為一個特定的安全身分)
- Run-as authentication (which is useful if one call should proceed with a different security identity)
- Java Authentication and Authorization Service (JAAS)
- Java EE container authentication (so you can still use Container Managed Authentication if desired)
- Kerberos
- Java Open Source Single Sign-On (JOSSO) *
- OpenNMS Network Management Platform *
- AppFuse *
- AndroMDA *
- Mule ESB *
- Direct Web Request (DWR) *
- Grails *
- Tapestry *
- JTrac *
- Jasypt *
- Roller *
- Elastic Path *
- Atlassian Crowd *
- Your own authentication systems (see below)

(標註 * 號代表由第三方提供)

Spring Security 也可以跟自定義的驗證機制做整合好棒棒。

不管是用哪一種驗證機制，Spring Security 都提供了許多授權功能，主要有三個重點：網路請求的授權、方法是否被觸發的授權以及接觸特定物件實例的授權。