---
title: "18.1. 總覽"
type: docs
---

# 18.1. 總覽

記得我或持續登入驗證是指網站能夠在 session 之間記得某個使用端的身份。一般來說是透過發送一個 cookie 到瀏覽器保存，然後在未來的 session 偵測到此 cookie 就可以自動登入。

Spring Security 提供了執行這個行動必要的觸發掛勾，並且有兩種記得我的實作。一種是利用雜湊來保存安全 token 到 cookie 中；另一種則是使用資料庫或其他持久保存機制來保存產生的 token。

注意這兩種實作都需要 `UserDetailsService`。如果你採用了一個不使用 `UserDetailsService` 的驗證提供者，例如 LDAP 提供者，那你必須要在 application context 中提供 `UserDetailsService` 的 bean，否這這兩種實作都無法運作。