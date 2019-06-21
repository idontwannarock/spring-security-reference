---
title: "15.1. `FilterSecurityInterceptor`"
type: docs
---

# 15.1. `FilterSecurityInterceptor`

我們在討論 [9.5. 存取控制 (授權管理)]({{< relref "/docs/9_5_acce_cont_auth.md" >}}) 的時候已經有簡單的看過 `FilterSecurityInterceptor`，而已經在命名空間使用 `<intercept-url>` 元素對它做設定。現在我們要來看如何仔細的設定它來給 `FilterChainProxy` 使用，順帶提到它的同伴 `ExceptionTranslationFilter`。

典型的設定範例如下：

```xml
<bean id="filterSecurityInterceptor" class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
<property name="authenticationManager" ref="authenticationManager"/>
<property name="accessDecisionManager" ref="accessDecisionManager"/>
<property name="securityMetadataSource">
	<security:filter-security-metadata-source>
	<security:intercept-url pattern="/secure/super/**" access="ROLE_WE_DONT_HAVE"/>
	<security:intercept-url pattern="/secure/**" access="ROLE_SUPERVISOR,ROLE_TELLER"/>
	</security:filter-security-metadata-source>
</property>
</bean>
```

`FilterSecurityInterceptor` 負責處理 HTTP 資源的安全，它需要 `AuthenticationManager` 及 `AccessDecisionManager` 的參考，它也提供針對不同 HTTP URL 請求的 configuration attributes，請參考 之前對於 [configuration attributes 的討論]({{< relref "/docs/9_5_acce_cont_auth.md#什麼是-configuration-attributes" >}})。

有兩種方式可以使用 configuration attributes 來對 `FilterSecurityInterceptor` 做設定。

第一種就是如上面程式碼所示，使用 `<filter-security-metadata-source>` 命名空間元素。這跟命名空間設定章節提到的 `<http>` 很類似，但 `<interceptor-url>` 子元素只使用 `pattern` 及 `access` 屬性，並用逗點分隔適用於個別 HTTP URL 的不同 configuration attributes。

第二種你必須撰寫你自己的 `SecurityMetadataSource`，但這已經超出這份文件的範圍。不過不論使用哪種方法，`SecurityMetadataSource` 都是負責用來回傳一個 `List<ConfigAttribute>`，這個列表包含所有關聯到一個已受保護的 HTTP URL 的所有 configuration attributes。

值得注意的是 `FilterSecurityInterceptor.setSecurityMetadataSource()` 方法其實預期接收一個 `FilterInvocationSecurityMetadataSource` 的實例當入參。`FilterInvocationSecurityMetadataSource` 是一個標記用介面，是 `SecurityMetadataSource` 的子介面，它只是用來表示 `SecurityMetadataSource` 了解 `FilterInvocation`。

為了方便講解，我們後面會用 `SecurityMetadataSource` 來指稱 `FilterInvocationSecurityMetadataSource`，因為兩者之間的差異對多數使用者來說都沒有關係。

透過將請求 URL 與設定好的 `pattern` 屬性比對，由命名空間語法建立的 `SecurityMetadataSource` 會取得對應特定的 `FilterInvocation` 的 configuration attributes，這個行為與以命名空間做設定是相同的。預設會將所有表達式當作 Apache Ant 風格路徑，也支援正則表達式以處理更複雜的狀況，`request-matcher` 屬性就是用來要使用怎樣的樣式。但是將在同一個定義當中，表達式是不能混用的。

如果將前一個範例寫成正則表達式則會如下：

```xml
<bean id="filterInvocationInterceptor"
	class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
<property name="authenticationManager" ref="authenticationManager"/>
<property name="accessDecisionManager" ref="accessDecisionManager"/>
<property name="runAsManager" ref="runAsManager"/>
<property name="securityMetadataSource">
	<security:filter-security-metadata-source request-matcher="regex">
	<security:intercept-url pattern="\A/secure/super/.*\Z" access="ROLE_WE_DONT_HAVE"/>
	<security:intercept-url pattern="\A/secure/.*\" access="ROLE_SUPERVISOR,ROLE_TELLER"/>
	</security:filter-security-metadata-source>
</property>
</bean>
```

比對動作會依照樣式設定的順序來進行，所以越特殊的樣式應該定義在更前面，如上面的範例所示。