---
title: "16.3. Servlet 3.1+ 整合"
---

# 16.3. Servlet 3.1+ 整合

這個段落會說明 Spring Security 整合 Sevlet 3.1 的方法。

## 16.3.1. `HttpServletRequest#changeSessionId()`

[`HttpServletRequest#changeSessionId()`](https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html#changeSessionId()) 是 Servlet 3.1 開始用來對付 [Session 固定攻擊](https://en.wikipedia.org/wiki/Session_fixation) 的預設方法。