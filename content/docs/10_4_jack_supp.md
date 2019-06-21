---
title: "10.4. Jackson 支援"
---

# 10.4. Jackson 支援

Spring Security 支援 Jackson 以將有關 Spring Security 的類別序列化，這樣在運作分散式 sessions (例如複製 sessions、Spring Session 等) 的時候，可以提升序列化 Spring Security 有關類別的效能。

要使用 Jackson，需要將 `JacksonJacksonModules.getModules(ClassLoader)` 登記為 [Jackson 模組](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/jackson2/SecurityJackson2Modules.html)。

```java
ObjectMapper mapper = new ObjectMapper();
ClassLoader loader = getClass().getClassLoader();
List<Module> modules = SecurityJackson2Modules.getModules(loader);
mapper.registerModules(modules);

// ... use ObjectMapper as normally ...
SecurityContext context = new SecurityContextImpl();
// ...
String json = mapper.writeValueAsString(context);
```