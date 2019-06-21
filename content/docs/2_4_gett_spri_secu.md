---
title: "2.4. 取得 Spring Security"
slug: "gett_spri_secu"
date: 2019-05-31T15:28:50+08:00
draft: false
type: docs
---

# 2.4. 取得 Spring Security

有好幾種取得 Spring Security 的方式，可以從 [Spring Security 的 Github](https://github.com/spring-projects/spring-security/releases) 下載打包好的發行版本、可以從 [Maven Central](https://mvnrepository.com/repos/central) 直接下載 jar 檔，也可以取得原始碼自己建立專案。

## 2.4.1. 使用 Maven

一組最基本的 Spring Security Maven 組合通常會長這樣：

**pom.xml**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.0.7.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>5.0.7.RELEASE</version>
    </dependency>
</dependencies>
```

如果有需要其他功能例如 LDAP、OpenID 等等，你會需要再加入 [其他模組](#2-4-3-專案模組)。

### Maven Repositories

所有正式發行的版本 (版本號後綴為 .RELEASE) 都會放上 Maven Central，所以不需要另外在 pom 檔宣告其他 Maven repositories。

但如果你使用的是 SNAPSHOT 版本，你就需要確保你有在 pom 檔有以下宣告：

```xml
<repositories>
    <repository>
        <id>spring-snapshot</id>
        <name>Spring Snapshot Repository</name>
        <url>http://repo.spring.io/snapshot</url>
    </repository>
</repositories>
```

如果你使用的是 milestone 或發行候選的版本，則需要以下宣告：

```xml
<repositories>
    <repository>
        <id>spring-milestone</id>
        <name>Spring Milestone Repository</name>
        <url>http://repo.spring.io/milestone</url>
    </repository>
</repositories>
```

### Spring 框架 Bom

Spring Security 是建立在 Spring Framework 5.0.x.RELEASE，但使用 4.0.x 的版本應該也可以。

有些使用者會碰到 Spring Security 在解析對 Spring Framework 5.0.8.RELEASE 的 [遞移依賴 (transitive dependencies)](http://blog.appx.tw/2017/03/28/maven-10-dependency-management/) 的時候，會碰到奇怪的 classpath 問題。

> 遞移依賴 (transitive dependencies) 是 Maven 的機制，意思是只要定義好某項 dependency，Maven 就會自動去查看該 dependency 的 pom 檔，並將其中所有關聯的 dependency 一併下載回來

其中一個比較醜的解決辦法，就是將所有 Spring Framework 模組的依賴都加到 `<dependencyManagement>` 段落中。

另一個方法是將 `spring-framework-bom` 加到 `<dependencyManagement>` 段落中，如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>5.0.x.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
	</dependencies>
</dependencyManagement>
```

這樣就可以確保 Spring Security 所有的遞移依賴都使用 Spring 5.0.x.RELEASE 模組。

> 另外一個注意事項，就是使用 bom 的方法只適用於 Maven 2.0.9 以上的版本，其他我不想多說，自己參考 [Maven 官網](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

## 2.4.2. Gradle

我沒有在用 Gradle，所以跳過。囧>

## 2.4.3. 專案模組

從 Spring Security 3.0 開始，程式碼就被分離成不同的 jar 檔，以便更清楚的區分功能區塊跟第三方依賴。

如果使用 Maven 來建立專案，那以下這些就是會加到你的 pom.xml 的模組；即使你不是用 Maven，我們都建議你看一下 pom 檔，這樣對於有用到那些第三方依賴跟版本會有些概念。

或者查看範例程式有包含哪些套件也是個好主意呀好主意~

### 核心 - spring-security-core.jar

包含核心的權限驗證及存取控制類別和介面、遠端支援及基本的設定 API，使用 Spring Security 必備。

支援獨立應用程式、遠端客戶端、方法 (服務層) 安全控制及 JDBC 使用者設定。

含有以下套件：

- `org.springframework.security.core`
- `org.springframework.security.access`
- `org.springframework.security.authentication`
- `org.springframework.security.provisioning`

### 遠端 - spring-security-remoting.jar

提供與 Spring Remoting 的整合，除非你在使用 Spring Remoting 開發遠端客戶端，否則你不需要這個部分。

主要的套件是 `org.springframework.security.remoting`。

### 網路 - spring-security-web.jar

包含 filters 及有關的網路安全架構程式碼，都依賴於 servlet API。

如果你要使用 Spring Security 網路權限驗證服務及 URL 存取控制，就需要它。

主要的套件是 `org.springframework.security.web`。

### 設定 - spring-security-config.jar

包含解析 security 命名空間及 Java 設定的程式碼。

如果你使用 Spring Security XML 命名空間或 Spring Security 支援的 Java 方式作設定，就會需要它。

主要套件是 `org.springframework.security.config`，但裡面所有類別都不應該在應用程式中直接使用。

### LDAP - spring-security-ldap.jar

使用 LDAP 驗證必備。

最高階層套件為 `org.springframework.security.ldap`。

### OAuth 2.0 Core - spring-security-oauth2-core.jar

使用 OAuth 2.0 或 OpenId Connect Core 1.0 如作為 Client、Resource 或 Authorization Server 的應用程式，Spring Security提供對 _OAuth 2.0 Authorization Framework_ 及 _OpenID Connect Core 1.0_ 支援的核心類別及介面。

最高階層套件為 `org.springframework.security.oauth2.core`

### OAuth 2.0 Client - spring-security-oauth2-client.jar

當應用程式利用 OAuth 2.0 Login 或 OAuth Client support，Spring Security 提供對 _OAuth 2.0 Authorization Framework_ 及 _OpenID Connect Core 1.0_ 客戶端的支援。

最高階層套件為 `org.springframework.security.oauth2.client`

### OAuth 2.0 JOSE - spring-security-oauth2-jose.jar

JOSE (Javascript Object Signing and Encryption) 框架的目的是為了提供一種能夠在端點間安全的傳送內文 (claim) 的方法，由以下規格集合所組成：

- JSON Web Token (JWT)
- JSON Web Signature (JWS)
- JSON Web Encryption (JWE)
- JSON Web Key (JWK)

Spring Security 對 JOSE 框架提供的支援，有以下最高層級的套件：

- `org.springframework.security.oauth2.jwt`
- `org.springframework.security.oauth2.jose`

### ACL - spring-security-acl.jar

沒在用先跳過。

### CAS - spring-security-cas.jar

沒在用先跳過。

### OpenID - spring-security-openid.jar

沒在用先跳過。

### Test - spring-security-test.jar

提供在 Spring Security 架構下的測試。

## 2.4.4. 原始碼

Spring Security 是一個開源的專案，我們鼓勵用 git 下載原始碼，這樣你就可以對它想幹嘛就幹嘛 (無誤)

用以下的 git 命令就可以取得整個專案的原始碼：

```
git clone https://github.com/spring-projects/spring-security.git
```

醬就可以取得整個專案所有的版本跟分支惹 <3