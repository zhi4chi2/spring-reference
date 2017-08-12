# Summary
## [Spring Framework](/framework/README.md)
- Core Technologies
  - The IoC container
    - Container overview
      - [Instantiating a container](/framework/beans/beans-factory-instantiation.md)
    - Bean overview
      - [Naming beans](/framework/beans/beans-beanname.md)
      - [Instantiating beans](/framework/beans/beans-factory-class.md)
    - Dependencies
      - [Dependency Injection](/framework/beans/beans-factory-collaborators.md)
      - [Dependencies and configuration in detail](/framework/beans/beans-factory-properties-detailed.md)
      - [Using depends-on](/framework/beans/beans-factory-dependson.md)
      - [Lazy-initialized beans](/framework/beans/beans-factory-lazy-init.md)
      - [Autowiring collaborators](/framework/beans/beans-factory-autowire.md)
- Data Access
  - [Data access with JDBC](/framework/jdbc/README.md)


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
