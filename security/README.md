- [Preface](/security/preface/README.md)
  - Getting Started
  - [Introduction](/security/preface/introduction.md)
  - [What's New in Spring Security 4.2](/security/preface/new.md)
  - [Samples and Guides](/security/preface/samples.md)
  - [Java Configuration](/security/preface/jc.md)
  - [Security Namespace Configuration](/security/preface/ns-config/README.md)
    - [A Minimal http Configuration](/security/preface/ns-config/minimal-http.md)
    - [form login](/security/preface/ns-config/form-login.md)
    - [basic login](/security/preface/ns-config/basic-login.md)
    - [logout](/security/preface/ns-config/logout.md)
    - [Using other Authentication Providers](/security/preface/ns-config/authentication-provider.md)
    - [Adding a Password Encoder](/security/preface/ns-config/password-encoder.md)
    - [Detecting Timeouts](/security/preface/ns-config/invalid-session-url.md)
    - [Concurrent Session Control](/security/preface/ns-config/concurrency-control.md)
  - [Sample Applications](/security/preface/sample-apps.md)
  - Spring Security Community
- [Architecture and Implementation](/security/overall-architecture/README.md)
  - [Technical Overview](/security/overall-architecture/technical-overview.md)
  - [Core Services](/security/overall-architecture/core-services.md)
- [Testing](/security/test/README.md)
- [Web Application Security](/security/web-app-security/README.md)
  - [The Security Filter Chain](/security/web-app-security/security-filter-chain.md)
  - [Core Security Filters](/security/web-app-security/core-web-filters.md)
  - [Servlet API integration](/security/web-app-security/servletapi.md)
  - [Remember-Me Authentication](/security/web-app-security/remember-me.md)
- [Authorization](/security/authorization/README.md)
  - [Authorization Architecture](/security/authorization/authz-arch/README.md)
- [Mine](/security/mine/README.md)
  - [输出用到的 beans](/security/mine/show-used-beans.md)


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


# Reference
- http://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/
- http://repo.spring.io/release/org/springframework/security/spring-security/4.2.3.RELEASE/spring-security-4.2.3.RELEASE-docs.zip 中有 pdf/epub 版本
