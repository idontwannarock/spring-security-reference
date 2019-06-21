---
title: "5.7. OAuth 2.0 登入"
type: docs
---

# 5.7. OAuth 2.0 登入

Spring Security 的 OAuth 2.0 Login 功能，讓應用程式可以提供使用者透過在 OAuth 2.0 Provider (例如 GitHub) 或 OpenID Connect 1.0 Provider (例如 Google) 已存在的帳號作登入。

OAuth 2.0 Login 功能實作了兩個使用案例：Google 登入或 GitHub 登入

> Spring Security 是以 **Authorization Code Grant** 實作 OAuth 2.0 Login 功能，參考 [OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749#section-4.1) 及 [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth)

## 5.7.1. Spring Boot 2.0 範例

Spring Boot 2.0 提供完整的 OAuth 2.0 Login 功能自動設定。

這個段落會示範如何以 Google 為 *Authentication Provider* 來設定 [OAuth 2.0 Login 功能範例](https://github.com/spring-projects/spring-security/tree/master/samples/boot/oauth2login)，並包含以下主題：

- [初始設定](#初始設定)
- [設定重新導向 URI](#設定重新導向-uri)
- [設定 `application.yml`](#設定-application-yml)
- [啟動應用程式](#啟動應用程式)

### 初始設定

要使用 Google 的 OAuth 2.0 驗證系統作登入，你必須要先在 Google API Console 新增一個 project 以取得 OAuth 2.0 的憑證 (credentials)。

> [Google 的 OAuth 2.0 驗證實作](https://developers.google.com/identity/protocols/OpenIDConnect)，符合 [OpenID Connect 1.0](https://openid.net/connect/) 的規範，而且該驗證是 [OpenID 憑證](https://openid.net/certification/)

請按照 [OpenID Connect](https://developers.google.com/identity/protocols/OpenIDConnect) 頁面 Setting up OAuth 2.0 這個段落的指示進行。

在完成 Obtain OAuth 2.0 credentials 段落的指示後，你應該就會獲得一個新的 OAuth Client，以及包含 Client ID 及 Client Secret 的憑證。

### 設定重新導向 URI

當終端使用者的使用者代理 (user-agent) 通過 Google 驗證並在同意畫面得到連接 OAuth Client 的許可後，被導向應用程式中的路徑，就是此處所說的重新導向的 URI。

在前段設定內的 Set a redirect URI 步驟，請確定 Authorized redirect URIs 欄位設定為 `http://localhost:8080/login/oauth2/code/google`。

> 預設的重新導向 URI 的樣版為 `{baseUrl}/login/oauth2/code/{registrationId}`，其中 **registrationId** 是給 [5.7.2. `ClientRegistration`](#5-7-2-clientregistration) 使用的獨特指示物

### 設定 `application.yml`

現在你已經從 Google 得到新的 OAuth Client，你還需要設定應用程式在驗證流程 (authentication flow) 去使用 OAuth Client：

1. 在 `application.yml` 作以下設定：

```yml
spring:
    security:
        oauth2:
            client:
                registration: # 1.
                    google:	  # 2.
                        client-id: google-client-id
                        client-secret: google-client-secret
```

- 1. `spring.security.oauth2.client.registration` 是 OAuth Client 屬性的基本前綴
- 2. 在前綴後則是 `ClientRegistration` 的 ID，例如 google

2. 將你前面取得的 OAuth 2.0 憑證取代 `client-id` 及 `client-secret` 的值

### 啟動應用程式

啟動 Spring Boot 2.0 範例然後前往 `http://localhost:8080`，你就會先被轉到預設自動產生的登入畫面，畫面會顯示一個 Google 的連結。

點擊該連結，就會被轉導到 Google 作驗證。

通過 Google 登入驗證後，會展示同意畫面詢問是否同意連接你前面建立的 OAuth Client，點擊允許以授權給 OAuth Client 取得如 email 等基本個人資訊。

在這個步驟，OAuth Client 會從 [UserInfo Endpoint](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) 取得你的 email 及基本資料，並建立一個已驗證 session。

## 5.7.2. `ClientRegistration`

`ClientRegistration` 是一個用 OAuth 2.0 或 OpenID Connect 1.0 Provider 註冊過的客戶端代表。

客戶端註冊會持有包括 client id、client secret、authorization grant type、redirect URI、scope(s)、authorization URI、token URI 等資訊。

`ClientRegistration` 及其屬性設定如下：

```java
public final class ClientRegistration {
	private String registrationId; // 1.
	private String clientId;	   // 2.
	private String clientSecret;   // 3.
	private ClientAuthenticationMethod clientAuthenticationMethod; // 4.
	private AuthorizationGrantType authorizationGrantType;         // 5.
	private String redirectUriTemplate;  // 6.
	private Set<String> scopes;          // 7.
	private ProviderDetails providerDetails;
	private String clientName;           // 8.

	public class ProviderDetails {
		private String authorizationUri; // 9.
		private String tokenUri;         // 10.
		private UserInfoEndpoint userInfoEndpoint;
		private String jwkSetUri;        // 11.

		public class UserInfoEndpoint {
			private String uri;                   // 12.
			private String userNameAttributeName; // 13.
		}
	}
}
```

1. `registrationId`: `ClientRegistration` 的獨特 ID 指示物
2. `clientId`: 客戶端的指示物
3. `clientSecret`: 客戶端的 secret
4. `clientAuthenticationMethod`: 用來跟 Provider 驗證 Client 的方法，支援的值為 **basic** 跟 **post**
5. `authorizationGrantType`: OAuth 2.0 授權框架定義了四種 [Authorization Grant](https://tools.ietf.org/html/rfc6749#section-1.3) 類型，支援的值為 **authorization_code** 跟 **implicit**
6. `redirectUriTemplate`: 在終端使用者通過驗證並獲得連接客戶端權限後，Authorization Server 會將終端使用者的使用者代理轉導向此客戶端登記的轉導 URI。預設的轉導 URI 樣版為 `{baseUrl}/login/oauth2/code/{registrationId}`，支援 URI 樣版變數
7. `scopes`: 在 Authorization Request 流程中，客戶端請求的範圍，例如 openid、email 或個人資料
8. `clientName`: 客戶端的描述性名稱，此名稱在某些特定的情景下可能會使用到，例如在自動產生的登入頁面顯示客戶端名稱
9. `authorizationUri`: Authorization Server 的 Authorization Endpoint URI
10. `tokenUri`: Authorization Server 的 Token Endpoint URI
11. `jwkSetUri`: 用來取得 Authorization Server 設定好的 [JSON Web Key (JWK)](https://tools.ietf.org/html/rfc7517)，JWK 內含用來驗證對應到 ID Token 或選用的 UserInfo Response 的 [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7517) 的加密密鑰
12. `(userInfoEndpoint)uri`: 用來連接經過驗證的終端使用者要求/屬性的 UserInfo Endpoint URI
13. `userNameAttributeName`: 對應終端使用者名稱或指示物的返回的 UserInfo Response 內含屬性的名稱

## 5.7.3. Spring Boot 2.0 屬性映射

以下表格表示 Spring Boot 2.0 OAuth Client 屬性與 `ClientRegistration` 屬性的對應關係。

|Spring Boot 2.0|`ClientRegistration`|
|---|---|
|`spring.security.oauth2.client.registration.[registrationId]`|`registrationId`|
|`spring.security.oauth2.client.registration.[registrationId].client-id`|`clientId`|
|`spring.security.oauth2.client.registration.[registrationId].client-secret`|`clientSecret`|
|`spring.security.oauth2.client.registration.[registrationId].client-authentication-method`|`clientAuthenticationMethod`|
|`spring.security.oauth2.client.registration.[registrationId].authorization-grant-type`|`authorizationGrantType`|
|`spring.security.oauth2.client.registration.[registrationId].redirect-uri-template`|`redirectUriTemplate`|
|`spring.security.oauth2.client.registration.[registrationId].scope`|`scopes`|
|`spring.security.oauth2.client.registration.[registrationId].client-name`|`clientName`|
|`spring.security.oauth2.client.provider.[providerId].authorization-uri`|`providerDetails.authorizationUri`|
|`spring.security.oauth2.client.provider.[providerId].token-uri`|`providerDetails.tokenUri`|
|`spring.security.oauth2.client.provider.[providerId].jwk-set-uri`|`providerDetails.jwkSetUri`|
|`spring.security.oauth2.client.provider.[providerId].user-info-uri`|`providerDetails.userInfoEndpoint.uri`|
|`spring.security.oauth2.client.provider.[providerId].userNameAttribute`|`providerDetails.userInfoEndpoint.userNameAttributeName`|

## 5.7.4. `ClientRegistrationRepository`

`ClientRegistrationRepository` 是作為 OAuth 2.0 / OpenID Connect 1.0 `ClientRegistration` 的 repository 來使用。

> 客戶端的註冊資訊是存放在相關聯的 Authorization Server 中，這個 repository 是用來取得存放在 Authorization Server 的主要客戶端註冊資訊的子集

Spring Boot 2.0 自動設定的功能會將 `spring.security.oauth2.client.registration.[registrationId]` 底下的屬性，與一個 `ClientRegistration` 的實例綁定，然後使用 `ClientRegistrationRepository` 將每個 `ClientRegistration` 的實例賦值。

> 預設的 `ClientRegistrationRepository` 實作為 `InMemoryClientRegistrationRepository`

自動設定功能同時也會在 `ApplicationContext` 中將 `ClientRegistrationRepository` 登記為一個 `@Bean`，以利應用程式需要的時候 DI。

以下是一個範例：

```java
@Controller
public class MainController {

	@Autowired
	private ClientRegistrationRepository clientRegistrationRepository;

	@RequestMapping("/")
	public String index() {
		ClientRegistration googleRegistration =
			this.clientRegistrationRepository.findByRegistrationId("google");

		...

		return "index";
	}
}
```

## 5.7.5. `CommonOAuth2Provider`

`CommonOAuth2Provider` 針對常用的 provider：Google、GitHub、Facebook 及 Okta，預先定義了一組預設的客戶端屬性。

例如 `authorization-uri`、`token-uri` 及 `user-info-uri` 在同一個 provider 裡不會經常改變，所以提供一組預設值來減少必須的設定也是很合理der。

如同前面的範例，當我們 [設定一組 Google 的客戶端](#設定-application-yml)，只有 `client-id` 及 `client-secret` 兩個屬性需要設定。

以下是一個範例：

```yml
spring:
    security:
        oauth2:
            client:
                registration:
                    google:
                        client-id: google-client-id
                        client-secret: google-client-secret
```

> 這裡的自動設定客戶端屬性預設值可以無縫接軌，是因為 registrationId (google) 與 `CommonOAuth2Provider` 內的枚舉常數 `GOOGLE` (大小寫沒差) 相符

某些情況下，你可能會想要指定一個不同的 `registrationId`，例如 `google-login`，你還是可以透過設定 `provider` 屬性來利用自動設定客戶端屬性預設值的功能。

例如：

```yml
spring:
    security:
        oauth2:
            client:
                registration:
                    google-login:        # 1.
                        provider: google # 2.
                        client-id: google-client-id
                        client-secret: google-client-secret
```

1. 將 `registrationId` 設為 `google-login`
2. 將 `provider` 屬性設為 `google`，如此就可以利用 `CommonOAuth2Provider.GOOGLE.getBuilder()` 的自動設定客戶端屬性預設功能

## 5.7.6. 設定自訂 Provider 屬性

有某些 OAuth 2.0 Provider 支援多重租賃技術，導致每個租戶 (或子網域) 會有不同的協定端點。

例如一個註冊了 Okta 的 OAuth Client 被分配到一個特定的子網域，並有自己的協定端點。

在這些情況下，Spring Boot 2.0 提供了以下的基本屬性，以自訂 provider 的屬性：`spring.security.oauth2.client.provider.[providerId]`。

例如：

```yml
spring:
    security:
        oauth2:
            client:
                registration:
                    okta:
                        client-id: okta-client-id
                        client-secret: okta-client-secret
                provider:
                    okta: # 1.
                        authorization-uri: https://your-subdomain.oktapreview.com/oauth2/v1/authorize
                        token-uri: https://your-subdomain.oktapreview.com/oauth2/v1/token
                        user-info-uri: https://your-subdomain.oktapreview.com/oauth2/v1/userinfo
                        user-name-attribute: sub
                        jwk-set-uri: https://your-subdomain.oktapreview.com/oauth2/v1/keys
```

1. `spring.security.oauth2.client.provider.okta` 基本屬性，可以用來自訂協定端點的位址

## 5.7.7. Override Spring Boot 2.0 自動設定

Spring Boot 2.0 用來支援 OAuth Client 的自動設定類別是 `OAuth2ClientAutoConfiguration`。

它會執行以下任務：

- 註冊一個以從設定好的 OAuth Client 屬性而來的 `ClientRegistration` 組成的 `ClientRegistrationRepository` `@Bean`
- 提供一個 `WebSecurityConfigurerAdapter` `@Configuration`，並透過 `httpSecurity.oauth2Login()` 開啟 OAuth 2.0 Login 功能

如果你有特定的需求要蓋過自動設定，你可以有以下幾種作法：

- [註冊一個 `ClientRegistrationRepository` `@Bean`](#註冊一個-clientregistrationrepository-bean)
- [提供一個 `WebSecurityConfigurerAdapter`](#提供一個-websecurityconfigureradapter)
- [完全 override 自動設定](#完全-override-自動設定)

### 註冊一個 `ClientRegistrationRepository` `@Bean`

以下是一個註冊 `ClientRegistrationRepository` `@Bean` 的範例：

```java
@Configuration
public class OAuth2LoginConfig {

	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	private ClientRegistration googleClientRegistration() {
		return ClientRegistration.withRegistrationId("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.clientAuthenticationMethod(ClientAuthenticationMethod.BASIC)
			.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
			.redirectUriTemplate("{baseUrl}/login/oauth2/code/{registrationId}")
			.scope("openid", "profile", "email", "address", "phone")
			.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth")
			.tokenUri("https://www.googleapis.com/oauth2/v4/token")
			.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo")
			.userNameAttributeName(IdTokenClaimNames.SUB)
			.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs")
			.clientName("Google")
			.build();
	}
}
```

### 提供一個 `WebSecurityConfigurerAdapter`

以下範例示範如何提供一個帶有 `@EnableWebSecurity` 註解的 `WebSecurityConfigurerAdapter`，並透過 `httpSecurity.oauth2Login()` 啟動 OAuth 2.0 登入功能：

```java
@EnableWebSecurity
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.oauth2Login();
	}
}
```

### 完全 override 自動設定

以下示範如何完全 override 自動設定，其實就是結合前兩個部分啦顆顆：

```java
@Configuration
public class OAuth2LoginConfig {

	@EnableWebSecurity
	public static class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeRequests()
					.anyRequest().authenticated()
					.and()
				.oauth2Login();
		}
	}

	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	private ClientRegistration googleClientRegistration() {
		return ClientRegistration.withRegistrationId("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.clientAuthenticationMethod(ClientAuthenticationMethod.BASIC)
			.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
			.redirectUriTemplate("{baseUrl}/login/oauth2/code/{registrationId}")
			.scope("openid", "profile", "email", "address", "phone")
			.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth")
			.tokenUri("https://www.googleapis.com/oauth2/v4/token")
			.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo")
			.userNameAttributeName(IdTokenClaimNames.SUB)
			.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs")
			.clientName("Google")
			.build();
	}
}
```

## 5.7.8. 如何不依靠 Spring Boot 2.0 來作 Java 設定

不多說，直接上範例：

```java
@Configuration
public class OAuth2LoginConfig {

	@EnableWebSecurity
	public static class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeRequests()
					.anyRequest().authenticated()
					.and()
				.oauth2Login();
		}
	}

	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	@Bean
	public OAuth2AuthorizedClientService authorizedClientService() {
		return new InMemoryOAuth2AuthorizedClientService(this.clientRegistrationRepository());
	}

	private ClientRegistration googleClientRegistration() {
		return CommonOAuth2Provider.GOOGLE.getBuilder("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.build();
	}
}
```

## 5.7.9. `OAuth2AuthorizedClient` / `OAuth2AuthorizedClientService`

`OAuth2AuthorizedClient` 代表一個已授權客戶端。只有當終端使用者 (資源擁有者) 授權客戶端連接受保護資源的情況下，該客戶端才會被視為已授權。

`OAuth2AuthorizedClient` 主要是用來將一個 `OAuth2AccessToken` 關連到一個 `ClientRegistration` 或資源擁有者身上，資源擁有者是指可以通過授權的 `Principal` 終端使用者。

而 `OAuth2AuthorizedClientService` 的主要腳色則是管理 `OAuth2AuthorizedClient` 實例。從開發者的角度來說，它提供了尋找關連到一個客戶端的 `OAuth2AccessToken` 的功能，這樣它才能被用來向資源伺服器發起請求。

> Spring Boot 2.0 自動設定功能會在 `ApplicationContext` 中註冊一個 `OAuth2AuthorizedClientService` `@Bean`

開發者也可以透過在 `ApplicationContext` 中註冊一個 `OAuth2AuthorizedClientService` `@Bean` 的方式來獲得尋找與特定 `ClientRegistration` 關聯的 `OAuth2AccessToken`。

範例：

```java
@Controller
public class MainController {

	@Autowired
	private OAuth2AuthorizedClientService authorizedClientService;

	@RequestMapping("/userinfo")
	public String userinfo(OAuth2AuthenticationToken authentication) {
		// authentication.getAuthorizedClientRegistrationId() returns the
		// registrationId of the Client that was authorized during the Login flow
		OAuth2AuthorizedClient authorizedClient =
			this.authorizedClientService.loadAuthorizedClient(
				authentication.getAuthorizedClientRegistrationId(),
				authentication.getName());

		OAuth2AccessToken accessToken = authorizedClient.getAccessToken();

		...

		return "userinfo";
	}
}
```

## 5.7.10. 其他資源

以下是一些描述更進階設定選項的資源：

- 31.1. OAuth 2.0 Login Page
- Authorization Endpoint:
    - 31.2.1. AuthorizationRequestRepository
- 31.3. Redirection Endpoint
- Token Endpoint:
    - 31.4.1. OAuth2AccessTokenResponseClient
- UserInfo Endpoint:
    - 31.5.1. Mapping User Authorities
    - 31.5.2. Configuring a Custom OAuth2User
    - 31.5.3. OAuth 2.0 UserService
    - 31.5.4. OpenID Connect 1.0 UserService