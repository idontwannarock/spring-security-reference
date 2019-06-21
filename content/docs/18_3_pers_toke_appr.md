---
title: "18.3. 持久化 Token 方式"
---

# 18.3. 持久化 Token 方式

這個方法是基於 Barry Jaspan 文章 Improved Persistent Login Cookie Best Practice (文章網址 http://jaspan.com/improved_persistent_login_cookie_best_practice 已失效) 及底下的討論，並作了一些小調整。

要使用命名方式來啟用這個方式，你需要提供 datasource 的參考：

```xml
<http>
    ...
    <remember-me data-source-ref="someDataSource"/>
</http>
```

而資料庫中應該要有一個名為 `persistent_logins` 的資料表，可以用以下或同等的 SQL 建立：

```sql
create table persistent_logins (
	username varchar(64) not null,
	series varchar(64) primary key,
	token varchar(64) not null,
	last_used timestamp not null)
```