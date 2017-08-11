# Summary
## [Spring Framework](/framework/README.md)
- Overview of Spring Framework
  - Getting Started with Spring
  - [Introduction to the Spring Framework](/framework/overview/README.md)
- What’s New in Spring Framework 4.x
  - [New Features and Enhancements in Spring Framework 4.0](/framework/new-in-4.0/README.md)
  - [New Features and Enhancements in Spring Framework 4.1](/framework/new-in-4.1/README.md)
  - [New Features and Enhancements in Spring Framework 4.2](/framework/new-in-4.2/README.md)
  - [New Features and Enhancements in Spring Framework 4.3](/framework/new-in-4.3/README.md)
- Core Technologies
  - [The IoC container](/framework/beans/README.md)
    - Introduction to the Spring IoC container and beans
    - [Container overview](/framework/beans/02.md)
    - [Bean overview](/framework/beans/03.md)
    - [Dependencies](/framework/beans/04.md)
    - [Bean scopes](/framework/beans/05.md)
    - [Customizing the nature of a bean](/framework/beans/06.md)
    - [Bean definition inheritance](/framework/beans/07.md)
    - [Container Extension Points](/framework/beans/08.md)
    - [Annotation-based container configuration](/framework/beans/09.md)
    - [Classpath scanning and managed components](/framework/beans/10.md)
    - [Using JSR 330 Standard Annotations](/framework/beans/11.md)
    - [Java-based container configuration](/framework/beans/12.md)
    - [Environment abstraction](/framework/beans/13.md)
    - [Registering a LoadTimeWeaver](/framework/beans/14.md)
    - [Additional Capabilities of the ApplicationContext](/framework/beans/15.md)
    - [The BeanFactory](/framework/beans/16.md)
- Data Access
  - [Data access with JDBC](/framework/jdbc/README.md)
    - [Introduction to Spring Framework JDBC](/framework/jdbc/01.md)
    - [Using the JDBC core classes to control basic JDBC processing and error handling](/framework/jdbc/02/README.md)
      - [JdbcTemplate](/framework/jdbc/02/01.md)
      - [NamedParameterJdbcTemplate](/framework/jdbc/02/02.md)


## [Spring Security](/security/README.md)
- Preface
  - Java Configuration
  - Security Namespace Configuration
    - [A Minimal http Configuration](/security/preface/ns-config/ns-minimal.md)
    - [Form Login Options](/security/preface/ns-config/ns-form.md)
    - [Basic Login Options](/security/preface/ns-config/ns-basic.md)
    - [Logout Handling](/security/preface/ns-config/ns-logout.md)
    - [Using other Authentication Providers](/security/preface/ns-config/ns-auth-providers.md)
    - [Adding a Password Encoder](/security/preface/ns-config/ns-password-encoder.md)
    - [Detecting Timeouts](/security/preface/ns-config/detecting-timeouts.md)
    - [Concurrent Session Control](/security/preface/ns-config/ns-concurrent-sessions.md)
- Architecture and Implementation
  - Technical Overview
    - [Localization](/security/overall-architecture/technical-overview/localization.md)
- Testing
- Web Application Security
  - Remember-Me Authentication
    - [Simple Hash-Based Token Approach](/security/web-app-security/remember-me/remember-me-hash-token.md)
- Mine
  - [输出用到的 beans](/security/mine/show-used-beans.md)


## [Spring Data](/data/README.md)
- [JPA](/data/jpa/README.md)
- [Spring Data 实战](/data/book/README.md)


## [Spring Boot](/boot/README.md)


## API
- org.springframework.context.support
  - [org.springframework.context.support.ReloadableResourceBundleMessageSource](/api/org/springframework/context/support/ReloadableResourceBundleMessageSource.md)
- org.springframework.security.web.authentication
  - [org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter](/api/org/springframework/security/web/authentication/UsernamePasswordAuthenticationFilter.md)
- org.springframework.security.web.authentication.logout
  - [org.springframework.security.web.authentication.logout.LogoutFilter](/api/org/springframework/security/web/authentication/logout/LogoutFilter.md)
- org.springframework.security.web.authentication.rememberme
  - [org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices](/api/org/springframework/security/web/authentication/rememberme/TokenBasedRememberMeServices.md)
- org.springframework.security.web.authentication.ui
  - [org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter](/api/org/springframework/security/web/authentication/ui/DefaultLoginPageGeneratingFilter.md)
