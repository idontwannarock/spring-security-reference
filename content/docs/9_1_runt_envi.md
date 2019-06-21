---
title: "9.1. 運行環境"
---

# 9.1. 運行環境

Spring Security 3.0 需要 JRE 5.0 或更高。

因為 Spring Security 目的是以自給自足的方式運行，所以並不需要在 JRE 放置任何特別的設定檔案，特別是不需要設定一個 [JAAS (Java Authentication and Authorization Service)](https://en.wikipedia.org/wiki/Java_Authentication_and_Authorization_Service) policy 檔，或將 Spring Security 放到常見的 classpath 位置。

就像使用 EJB 或 Servlet 容器，你並不需要在哪裡放特別的設定檔，也不需要在 server 的 classloader 載入 Spring Security。

這樣的設計最大程度的提供佈署的彈性，只要將目標檔案 (例如 JAR、WAR、EAR) 複製到目的地就可以立刻運作。