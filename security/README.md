# 术语
- principal - generally means a user, device or some other system which can perform an action in your application
- authentication entry point - 当没有认证的用户访问受保护资源时，认证过程被触发的地方。


预留的 URL
- AbstractAuthenticationProcessingFilter.requiresAuthenticationRequestMatcher - 默认 /login & method="POST" 参见 AuthenticationConfigBuilder 第 209 行。其中 method="POST" 不可设置。 /login 可以通过 form-login.login-processing-url 设置，参见 FormLoginBeanDefinitionParser 第 175 行
  - UsernamePasswordAuthenticationFilter.usernameParameter - 默认 username 参见 UsernamePasswordAuthenticationFilter 第 54 行。可以通过 form-login.username-parameter 设置，参见 FormLoginBeanDefinitionParser 第 137 行
  - UsernamePasswordAuthenticationFilter.passwordParameter - 默认 password 参见 UsernamePasswordAuthenticationFilter 第 55 行。可以通过 form-login.password-parameter 设置，参见 FormLoginBeanDefinitionParser 第 141 行
  - AbstractRememberMeServices.parameter - 默认 remember-me 参见 AbstractRememberMeServices 第 80 行。可以通过 remember-me.remember-me-parameter 设置，参见 RememberMeBeanDefinitionParser 第 161 行
- DefaultLoginPageGeneratingFilter.loginPageUrl - 默认 /login & method="GET" 参见 DefaultLoginPageGeneratingFilter 第 82, 282 行。在 AuthenticationConfigBuilder 第 541 行创建 DefaultLoginPageGeneratingFilter 时没有设置 loginPageUrl ，所以 /login & method="GET" 都是不可以设置的。
- LogoutFilter.logoutRequestMatcher - 默认 /logout & method="POST" 参见 LogoutBeanDefinitionParser 第 42, 133 行。其中 method 是由 csrf 决定的，不可设置。 /logout 可以通过 logout.logout-url 设置，参见 LogoutBeanDefinitionParser 第 87 行。


预留的 bean
- springSecurityFilterChain - org.springframework.security.config.BeanIds.SPRING_SECURITY_FILTER_CHAIN


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


Maven repo 中的其它 artifact
- spring-security-crypto
- spring-security-taglibs
- spring-security-aspects
- spring-security-data - 4.0 新增
- spring-security-messaging - 4.0 新增
- spring-security-samples-javaconfig-messages - 不在 BOM 中， 4.1.0.RELEASE 新增
- spring-security-samples-messages-jc - 不在 BOM 中，最新版 4.0.4.RELEASE


# 攻击保护


# Reference
- http://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/
- http://repo.spring.io/release/org/springframework/security/spring-security/4.2.3.RELEASE/spring-security-4.2.3.RELEASE-docs.zip 中有 pdf/epub 版本
