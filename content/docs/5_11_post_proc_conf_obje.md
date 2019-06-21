---
title: "5.11. 已設定物件的後處理"
type: docs
---

# 5.11. 已設定物件的後處理

Spring Security 的 Java 設定機制並不會將其設定的每個物件的每個屬性都暴露出來，這樣可以簡化絕大多數使用者的設定，畢竟如果所有屬性都暴露出來，那乾脆直接用標準 bean 去設定就好。

雖然有很好的理由不直接將每條屬性都暴露出來，但使用者可能還是需要能更進一步作設定的選項。

為了處理這個部分，Spring Security 引進了 `ObjectPostProcessor` 的概念，它可以用來修改或取代許多由 Java 設定機制建立的物件實例。

例如如果你想要對 `FilterSecurityInterceptor` 的 `filterSecurityPublishAuthorizationSuccess` 屬性作設定，你可以這樣做：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.anyRequest().authenticated()
			.withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
				public <O extends FilterSecurityInterceptor> O postProcess(O fsi) {
					fsi.setPublishAuthorizationSuccess(true);
					return fsi;
				}
			});
}
```