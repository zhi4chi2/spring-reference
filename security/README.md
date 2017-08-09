- [Spring Security](/security/README.md)
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
    - [Remember-Me Authentication](/security/web-app-security/remember-me.md)


# 术语
- principal - generally means a user, device or some other system which can perform an action in your application
- authentication entry point - 当没有认证的用户访问受保护资源时，认证过程被触发的地方。


预留的 URL
- /login
- /logout


预留的 bean
- springSecurityFilterChain


# Reference
- http://docs.spring.io/spring-security/site/docs/4.2.3.RELEASE/reference/htmlsingle/
- http://repo.spring.io/release/org/springframework/security/spring-security/4.2.3.RELEASE/spring-security-4.2.3.RELEASE-docs.zip 中有 pdf/epub 版本
