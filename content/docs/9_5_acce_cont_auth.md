---
title: "9.5. 存取控制 (授權管理)"
type: docs
---

# 9.5. 存取控制 (授權管理)

在 Spring Security 中最主要負責存取控制的介面就是 `AccessDecisionManager`。

在介面中有一個 `decide()` 方法會使用 `Authentication` 物件來請求存取 secure object 及用來申請該物件的 security metadata 屬性清單(例如腳色清單)。

## 9.5.1. Security 及 AOP Advice

需對 AOP 有概念，可參考 [AOP 及 Spring AOP 簡述](https://idontwannarock.github.io/tech_blog/2018/04/aop-and-spring-aop-basic/)

基本上 Advice 的位置 (JointPoint 連接點) 分為 before、after、throws 及 around。其中 around advice 相當有用，因為使用它可以選擇是否要繼續觸發方法、是否要變更回應或是否要丟出例外。

所以 Spring Security 使用 Spring 支援標準 AOP 的 around advice 來處理方法的觸發；並使用 filter 模擬 around advice 來處理 web 請求。

> 現代的 Java EE 應用程式通常都在 service layer 處理核心商業邏輯，所以很多人對如何在 service layer 處理方法的觸發比較關注，那 Spring AOP 就足夠了；如果你需要保護 domain object，那可以考慮 AspectJ  
> 所以大致上就是用 Spring AOP 保護 service layer 的方法觸發；AspectJ 保護 domain object；filter 處理 web 請求  
> 當然這三種都可以混用

## 9.5.2. Secure Object 及 `AbstractSecurityInterceptor`

Spring Security 定義 secure object 為任何允許安全控制行為 (例如資源控管) 的物件，例如方法觸發或 web 請求的對象。

每個型態的 secure object 都有對應的 interceptor 類別，這些 interceptor 都會繼承 `AbstractSecurityInterceptor`。

但重要的是當 `AbstractSecurityInterceptor` 被呼叫的時候，`SecurityContext` 已經持有合格的 `Authentication`，也就是該 principal 已經通過驗證。

`AbstractSecurityInterceptor` 提供的運作流程，通常是這樣：

1. 尋找跟目前請求有關聯的 configuration attributes
2. 提交該 secure object、當前的 `Authentication` 及 configuration attributes 給 `AccessDecisionManager` 以做出是否授權通過的決定
3. 可以自訂是否改變 secure object 觸發過程所需要的 `Authentication`
4. 在授權通過的前提下，允許 secure object 開始觸發過程
5. 如果 `AfterInvocationManager` 有被設定的話，則當觸發過程結束(return)後呼叫 `AfterInvocationManager`；但若是觸發過程產生例外，則 `AfterInvocationManager` 不會被呼叫

![AbstractSecurityInterceptor 與有關的物件圖示](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/images/security-interception.png "AbstractSecurityInterceptor 與有關的物件圖示")

### 什麼是 Configuration Attributes

一條 configuration attribute 可以被視為是一個對那些被 `AbstractSecurityInterceptor` 調用的類別有特殊意義的字串，可能單純是一個腳色名稱或有更複雜的意義，就看 `AccessDecisionManager` 怎麼被實作。

`SecurityMetadataSource` 是用來查看有關 secure object 的屬性，而 `AbstractSecurityInterceptor` 會依照 `SecurityMetadataSource` 被設定。

通常使用者是看不到這些設定的，configuration attributes 可能出現在被保護的方法的註解，或作為被保護的 URL 的 access attributes。

例如這樣：

```xml
<intercept-url pattern='/secure/**' access='ROLE_A,ROLE_B'/>
```

這意思是說 configuration attributes 為 `ROLE_A` 跟 `ROLE_B`，適用於符合 `/secure/**` 這種樣式的 web 請求。

在實作上以預設的 `AccessDecisionManager` 來說，任何持有著符合這兩個 attributes 當中任一個的 `GrantedAuthority` 的使用端，都會通過授權。

嚴格來說，這些 attributes 會被怎麼認定，必須要看 `AccessDecisionManager` 是怎麼被實作的。

前綴 ROLE_ 只是個標記，用來表示這些 attributes 代表腳色而且應該被 `RoleVoter` 取用，當然這只在使用 voter-based 的 `AccessDecisionManager` 的情況下才有關係。

### `RunAsManager`

假如 `AccessDecisionManager` 通過請求，通常 `AbstractSecurityInterceptor` 就會直接繼續處理該請求，但當有使用者需要替換 `SecurityContext` 裡面的 `Authentication`，就會由 `AccessDecisionManager` 呼叫一個 `RunAsManager` 來處理。

一般來說很少會有這種情況，但假如某個 service layer 的方法需要用一個不同的 identity 去呼叫某個遠端的系統，可是因為 Spring Security 預設會自動傳遞 security identity 到其他 server，這種情況下，`RunAsManager` 這種方式可能就會有用。

### `AfterInvocationManager`

當 secure object 觸發過程被執行而且結束(return)，不論是完成一個方法或或繼續執行 filter chain 的下一步，`AbstractSecurityInterceptor` 都會有一個最後的機會來操作這個觸發過程。

在這個階段，`AbstractSecurityInterceptor` 可能只會修改回傳的物件。

> 當授權無法在進入到 secure object 觸發過程前作出決定，那我們可能就會需要這樣做

如果有這樣的需求，`AbstractSecurityInterceptor` 會將控制權轉移給 `AfterInvocationManager` 去實際做變更、取代、丟出例外或不做變更，但這只有在觸發過程成功的前提下才會發生，一旦觸發過程丟出例外，這個部分就會被跳過。

### 延伸 Secure Object 模型

只有當開發人員需要新的 intercept 及 authorize 請求方式的時候，才有可能直接使用到 secure object。

例如要保護對 message 系統的呼叫，可能會需要建立一個新的 secure object。

也就是說，任何需要保護並提供監聽 intercept 呼叫方式，理論上都可以成為一個 secure object。

話雖這麼說，但多數的 Spring 應用程式都只會使用三種現有支援的 secure object，包括對應 AOP Alliance 的 `MethodInvocation`、AspectJ 的 `JoinPoint` 及 web 請求的 `FilterInvocation`。