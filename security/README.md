# 术语
- principal - generally means a user, device or some other system which can perform an action in your application
- authentication entry point - where the authentication process is triggered by an attempt by an unauthenticated user to access to a secured resource
- UserDetails - represents a principal, but in an extensible and application-specific way. adapter between your own user database and what Spring Security needs
- GrantedAuthority - an authority that is granted to the principal. application-wide permissions. not specific to a given domain object.
- authentication mechanism - e.g. form-base login(UsernamePasswordAuthenticationFilter)
- secure object - refer to any object that can have security (such as an authorization decision) applied to it. The most common examples are method invocations(MethodInvocation) and web requests(FilterInvocation).
- configuration attributes(ConfigAttribute)/security metadata attributes - may be simple role names or have more complex meaning, will be entered as annotations on secured methods or as access attributes on secured URLs.


authentication mechanism
- collecting authentication details from a user agent (usually a web browser)
- an Authentication "request" object is built and then presented to the AuthenticationManager.
  - receives back the fully-populated Authentication object, put the Authentication into the SecurityContextHolder, and cause the original request to be retried,
  - If the AuthenticationManager rejected the request, the authentication mechanism will ask the user agent to retry.


# Reserved
保留的 URL
- AbstractAuthenticationProcessingFilter.requiresAuthenticationRequestMatcher - 默认 /login & method="POST" 参见 AuthenticationConfigBuilder 第 209 行。其中 method="POST" 不可设置。 /login 可以通过 form-login.login-processing-url 设置，参见 FormLoginBeanDefinitionParser 第 175 行
  - UsernamePasswordAuthenticationFilter.usernameParameter - 默认 username 参见 UsernamePasswordAuthenticationFilter 第 54 行。可以通过 form-login.username-parameter 设置，参见 FormLoginBeanDefinitionParser 第 137 行
  - UsernamePasswordAuthenticationFilter.passwordParameter - 默认 password 参见 UsernamePasswordAuthenticationFilter 第 55 行。可以通过 form-login.password-parameter 设置，参见 FormLoginBeanDefinitionParser 第 141 行
  - AbstractRememberMeServices.parameter - 默认 remember-me 参见 AbstractRememberMeServices 第 80 行。可以通过 remember-me.remember-me-parameter 设置，参见 RememberMeBeanDefinitionParser 第 161 行
- DefaultLoginPageGeneratingFilter.loginPageUrl - 默认 /login & method="GET" 参见 DefaultLoginPageGeneratingFilter 第 82, 282 行。在 AuthenticationConfigBuilder 第 541 行创建 DefaultLoginPageGeneratingFilter 时没有设置 loginPageUrl ，所以 /login & method="GET" 都是不可以设置的。
- LogoutFilter.logoutRequestMatcher - 默认 /logout & method="POST" 参见 LogoutBeanDefinitionParser 第 42, 133 行。其中 method 是由 csrf 决定的，不可设置。 /logout 可以通过 logout.logout-url 设置，参见 LogoutBeanDefinitionParser 第 87 行。


保留的 bean name
- springSecurityFilterChain - org.springframework.security.config.BeanIds.SPRING\_SECURITY\_FILTER\_CHAIN


# Modules
- spring-security-core.jar - core authentication and access-contol classes and interfaces, remoting support and basic provisioning APIs. Supports standalone applications, remote clients, method (service layer) security and JDBC user provisioning(使用 JDBC 取得用户信息).
- spring-security-remoting.jar - 与 Spring Remoting 集成。当使用 Spring Remoting 编写 remote client 时使用
- spring-security-web.jar - filters and related web-security infrastructure code, 基于 URL 的访问控制，基于 web 的认证
- spring-security-config.jar - 解析 security 命名空间，其中的类不应该在 application 中直接使用
- spring-security-ldap.jar - LDAP 认证和管理 LDAP 条目
- spring-security-acl.jar - domain object ACL
- spring-security-cas.jar - 使用 CAS 认证
- spring-security-openid.jar - OpenID 认证，通过外部 OpenID server 认证用户。需要 OpenID4Java
- spring-security-test.jar - Support for testing with Spring Security, 4.0 新增


BOM 中的其它 artifact
- spring-security-crypto
- spring-security-taglibs
- spring-security-aspects
- spring-security-data - 4.0 新增
- spring-security-messaging - 4.0 新增


Maven repo 中的其它 artifact(不在 BOM 中)
- spring-security-samples-javaconfig-messages - 4.1.0.RELEASE 新增， 4.0.4.RELEASE 之前叫 spring-security-samples-messages-jc


# 攻击及保护
- cross-site scripting
- request-forgery
- session-hijacking
- man-in-the-middle attacks - HTTPS
- session fixation attacks - `<session-management>.session-fixation-protection`


# 测试项目
pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.example.demo</groupId>
    <artifactId>demo-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>

  <artifactId>demo-spring-security</artifactId>
  <packaging>war</packaging>

  <name>demo-spring-security</name>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-bom</artifactId>
        <version>4.2.3.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.taglibs</groupId>
      <artifactId>taglibs-standard-spec</artifactId>
      <version>1.2.5</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.taglibs</groupId>
      <artifactId>taglibs-standard-impl</artifactId>
      <version>1.2.5</version>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-acl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
    </dependency>

    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```


src/main/webapp/WEB-INF/web.xml
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID"
  version="2.5">
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring-*.xml</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```


src/main/resources/logback.xml
```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>[%thread] %-5level %logger- %msg%n</pattern>
    </encoder>
  </appender>

  <root level="info">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```


# Reference
- http://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/
- http://repo.spring.io/release/org/springframework/security/spring-security/4.2.3.RELEASE/spring-security-4.2.3.RELEASE-docs.zip 中有 pdf/epub 版本
