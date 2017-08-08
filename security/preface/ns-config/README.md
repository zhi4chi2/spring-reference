# Introduction
使用 security 命名空间（注意，需要 classpath 中有 spring-security-config.jar ）
```xml
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:security="http://www.springframework.org/schema/security"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
</beans>
```
或
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
</beans:beans>
```


## Design of the Namespace
namespace 分成几部分：
- Web/HTTP Security - Sets up the filters and related service beans used to apply the framework authentication mechanisms, to secure URLs, render login and error pages and much more.
- Business Object (Method) Security - 保护 service 层
- AuthenticationManager - 处理 authentication 请求
- AccessDecisionManager - provides access decisions for web and method security
- AuthenticationProviders - authentication manager 使用它们来认证用户
- UserDetailsService


# Getting Started with Security Namespace Configuration
## web.xml Configuration
在 web.xml 中配置过滤器
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID"
  version="2.5">
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


springSecurityFilterChain 是 namespace 创建的 bean 名。 DelegatingFilterProxy 将 filter 的方法委托给这个 bean 。


springSecurityFilterChain 这个名字定义在 org.springframework.security.config.BeanIds.SPRING_SECURITY_FILTER_CHAIN


## A Minimal &lt;http> Configuration
最简单的例子：
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
    <logout />
  </http>
</beans:beans>
```


上面的配置表示：
- 所有(/**)的 URL 都需要被保护，需要 ROLE_USER 角色
- form-login - 使用 form 方式登录 application
- logout - 注册一个 logout 的 URL ，可以登出 application


&lt;http> 是所有 web-related 元素的父元素。


&lt;http> 用于创建 FilterChainProxy 及其使用的 filter beans ，并且预定义了 filter beans 的顺序。


intercept-url.pattern 默认使用 ant path style syntax ，还可以配置使用 regular-expression 。


intercept-url.access 是逗号分隔的角色的列表。如果用户有其中一个 ROLE 则可以 make the request 。


如果没有定义 intercept-url.method ，则 Spring Security 会依次（按照顺序）查看是否符合 intercept-url.pattern ，使用第一个符合的 intercept-url.access 。
如果定义了 intercept-url.method ，则 Spring Security 在比对 URL 时，先查看完全符合的 intercept-url （包括 pattern 和 method ），如果有完全符合的，不论其顺序。


添加测试用户：
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
    <logout />
  </http>

  <authentication-manager>
    <authentication-provider>
      <user-service>
        <user name="jimi" password="jimispassword" authorities="ROLE_USER, ROLE_ADMIN" />
        <user name="bob" password="bobspassword" authorities="ROLE_USER" />
      </user-service>
    </authentication-provider>
  </authentication-manager>
</beans:beans>
```


- &lt;authentication-manager> 元素创建了一个 ProviderManager 对象。
- &lt;authentication-provider> 元素创建了一个 DaoAuthenticationProvider bean 。如果有多个 &lt;authentication-provider> 元素表示不同的 authentication sources ，则会依次使用。
- &lt;user-service> 元素创建了 InMemoryDaoImpl 对象。


还可以使用 user-service 元素的 properties 属性从 properties 文件中加载用户信息。


完整例子参见 [A Minimal http Configuration](/security/preface/namespace/minimal-http.md)


## Form and Basic Login Options
如果没有显式定义 form-login.login-page 属性，则生成登录页面（参见 org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter ），生成的登录页面大致如下：
```html
<html>
<head>
<title>Login Page</title>
</head>
<body onload='document.f.username.focus();'>
  <h3>Login with Username and Password</h3>
  <form name='f' action='/demo-spring-security/login' method='POST'>
    <table>
      <tr>
        <td>User:</td>
        <td><input type='text' name='username' value=''></td>
      </tr>
      <tr>
        <td>Password:</td>
        <td><input type='password' name='password' /></td>
      </tr>
      <tr>
        <td colspan='2'><input name="submit" type="submit" value="Login" /></td>
      </tr>
      <input name="_csrf" type="hidden" value="4a6d3b25-e422-4752-af28-14b815197c2f" />
    </table>
  </form>
</body>
</html>
```


可以指定自己的登录页面：
```xml
  <http>
    <intercept-url pattern="/login.jsp*" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" />
    <logout />
  </http>
```
注意：需要添加一个 intercept-url ，使得 login.jsp 对所有人都可见。否则将无法登录， SpringSecurity 在启动时会报 WARN


form-login 的属性
- default-target-url - 主动进入登录页登录，登录后的转向。默认是 "/"
- always-use-default-target
- authentication-success-handler-ref - AuthenticationSuccessHandler 的实现，替代 default-target-url


从 3.1 开始，可以配置多个 http 元素，也就是说可以对不同的资源使用完全不同的过滤器链。


using the attribute filters="none" on an intercept-url element is incompatible with this change and is no longer supported in 3.1.


对静态资源，不经过(bypass)过滤器链：
```xml
  <http pattern="/css/**" security="none" />
```


如果没有指定 http.pattern ，则表示匹配所有的请求（可以再使用 http.intercept-url 指定匹配的请求）


http.security="none" 和 intercept-url.access="IS_AUTHENTICATED_ANONYMOUSLY" 的区别：
- http.security="none" 完全没有安全过滤器链（空链），因此不能取得当前用户信息，不能调用安全方法(secured methods)
- intercept-url.access="IS_AUTHENTICATED_ANONYMOUSLY" 则可以


完整例子参见 [form login](/security/preface/namespace/form-login.md)


使用 basic 认证
```xml
  <http use-expressions="false">
    <intercept-url pattern="/**" access="ROLE_USER" />
    <http-basic />
  </http>
```
此时将优先使用 basic 认证。文档说这种方式时依然可以使用 form-login 但测试好像不可以。


完整例子参见 [basic login](/security/preface/namespace/basic-login.md)


## Logout Handling
参看 org.springframework.security.web.authentication.logout.LogoutFilter 。


```xml
    <logout />
```


属性：
- logout-url - 登出的 url 。缺省是 /logout ，参见 LogoutFilter 第 76 行
- logout-success-url - 登出后的 url 。默认是 form-login.login-page?logout 例如 "/login?logout" 参见 org.springframework.security.config.annotation.web.configurers.LogoutConfigurer 第 70 行
- success-handler-ref


完整例子参见 [Logout Handling](/security/preface/namespace/logout.md)


## Using other Authentication Providers
例如：
```xml
  <authentication-manager>
    <authentication-provider user-service-ref='myUserDetailsService' />
  </authentication-manager>
```
UserDetailsService 的实现可以从数据库或者其他地方(例如 LDAP)读取验证信息。


使用标准的 user 表（ #db_schema_users_authorities ）的 UserDetailsService 如下：
```xml
  <authentication-manager>
    <authentication-provider user-service-ref='myUserDetailsService' />
  </authentication-manager>

  <beans:bean id="myUserDetailsService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <beans:property name="dataSource" ref="dataSource" />
  </beans:bean>
```
或者
```xml
  <authentication-manager>
    <authentication-provider>
      <jdbc-user-service data-source-ref="dataSource" />
    </authentication-provider>
  </authentication-manager>
```


还可以自定义 AuthenticationProvider 
```xml
  <authentication-manager>
    <authentication-provider ref='myAuthenticationProvider' />
  </authentication-manager>
```
可以有多个 authentication-provider 元素，会依次轮询。


### Adding a Password Encoder
使用 &lt;authentication-provider>.&lt;password-encoder> 指定密码加密算法，使用 &lt;password-encoder>.&lt;salt-source> 指定盐值。
例如
```xml
      <password-encoder hash="sha">
        <salt-source user-property="username" />
      </password-encoder>
```
user-property 表示 UserDetails 对象的属性。


还可以自定义一个实现 PasswordEncoder 接口的 bean ，然后 &lt;password-encoder ref="" > 。例如
```xml
  <beans:bean name="bcryptEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />

  <authentication-manager>
    <authentication-provider>
      <password-encoder ref="bcryptEncoder" />
      <user-service>
        <user name="jimi" password="d7e6351eaa13189a5a3641bab846c8e8c69ba39f" authorities="ROLE_USER, ROLE_ADMIN" />
        <user name="bob" password="4e7421b1b8765d8f9406d87e7cc6aa784c4ab97f" authorities="ROLE_USER" />
      </user-service>
    </authentication-provider>
  </authentication-manager>
```


推荐使用 Bcrypt 而不是简单的 Hash 算法(SHA/MD5?)


完整例子参见 [Adding a Password Encoder](/security/preface/namespace/password-encoder.md)


# Advanced Web Features
## Remember-Me Authentication
参见 [Remember-Me Authentication](/security/web-app-security/remember-me/README.md)


## Adding HTTP/HTTPS Channel Security
如果应用同时支持 HTTP 和 HTTPS 协议，要求特定的 URL 只能使用 HTTPS ，可以直接使用 &lt;intercept-url> 的 requires-channel 属性：
```xml
  <http>
    <intercept-url pattern="/secure/**" access="ROLE_USER" requires-channel="https" />
    <intercept-url pattern="/**" access="ROLE_USER" requires-channel="any" />
  </http>
```
使用了这个配置以后，如果用户通过 HTTP 尝试访问 "/secure/**" 匹配的网址，他们会先被重定向到 HTTPS 网址下。参看 ChannelProcessingFilter 。


requires-channel 可用的值有：
- http
- https
- any


如果应用使用的不是 HTTP 或 HTTPS 的标准端口，可以用下面的方式指定端口对应关系：
```xml
  <http>
    <port-mappings>
      <port-mapping http="9080" https="9443" />
    </port-mappings>
  </http>
```


使用 HTTPS 可以防止 man-in-the-middle attacks


## Session Management
### Detecting Timeouts
如果用户提交了一个无效的(invalid) session ID ，可以配置 Spring Security redirect 到某个地址：
```xml
  <http>
    <session-management invalid-session-url="/invalidSession.htm" />
  </http>
```


注意，如果用户 logout 后没有关闭浏览器就又 login ，则上面的配置将会错误地转向 /invalidSession.htm 页面。因为 invalidate session(logout) 时没有清理 session cookie ，再次 login 时就会将此 cookie 发送。解决的办法是：
```xml
<logout delete-cookies="JSESSIONID" />
```
但是，这种解决办法不能保证每种 servlet container 都会起作用。


如果 application 在 proxy server 之后，还要配置 proxy server 删除 session cookie ，例如配置 Apache HTTPD 的 mod_headers 。 FIXME


完整例子参见 [Detecting Timeouts](/security/preface/namespace/invalid-session-url.md)


### Concurrent Session Control
首先需要在 web.xml 中添加
```xml
  <listener>
    <listener-class>
      org.springframework.security.web.session.HttpSessionEventPublisher
    </listener-class>
  </listener>
```


然后配置：
```xml
  <http>
    <session-management>
      <concurrency-control max-sessions="1" />
    </session-management>
  </http>
```
使得后一次登录踢掉前一次登录。


或者使用
```xml
  <session-management>
    <concurrency-control max-sessions="1" error-if-maximum-exceeded="true" />
  </session-management>
```
阻止第二次登录：
- 如果配置了 session-management.session-authentication-error-url ，则使用这个错误页面，否则
- 如果第二次登录是通过 form-login 登录的，则转向 authentication-failure-url
- 如果第二次登录不是交互式的，比如是通过 remember-me 登录的，则返回 401(unauthorized) 错误


如果你为 form-login 自定义了 authentication filter ，则必须显式配置 concurrent session control support 。


完整例子参见 [Concurrent Session Control](/security/preface/namespace/concurrency-control.md)


### Session Fixation Attack Protection
;Session fixation attacks: 黑客首先访问站点创建一个 session ，然后把带有 session 的链接发给用户，诱使他用此 session 登录。


Spring Security 为防止 Session fixation attack ，在每个用户登录时，创建一个新的 session 或者改变 session ID 。如果不希望如此，可以改变 session-management 元素的 session-fixation-protection 属性：
- none - 仍然使用原来的 session
- newSession - 创建一个新的 session ，但不拷贝旧 session 中的 attributes (Spring Security-related attributes 仍然会 copy)
- migrateSession - 创建一个新的 session ，并把旧的 session 中的所有 attribute 拷贝到新的 session 。使用 Servlet container 3.0- (含 3.0)时， Spring Security 使用它作为默认值。
- changeSessionId - 使用 Servlet container 提供的 session fixation protection (HttpServletRequest.changeSessionId()) ，这个方法从 Servlet 3.1 (Java EE 7) 起才可用。使用 Servlet container 3.1+ 时， Spring Security 使用它作为默认值。


如果发生 session fixation protection ，会触发 application context 的 SessionFixationProtectionEvent 事件。如果使用 changeSessionId ，还会通知 javax.servlet.http.HttpSessionIdListener 监听器。


## OpenID Support
OpenID 可以代替 form-based login ，也可以与 form-based login 一起使用。


```xml
  <http>
    <intercept-url pattern="/**" access="ROLE_USER" />
    <openid-login />
  </http>
  <authentication-manager>
    <authentication-provider>
      <user-service>
        <user name="http://jimi.hendrix.myopenid.com/" authorities="ROLE_USER" />
      </user-service>
    </authentication-provider>
  </authentication-manager>
```


http://jimi.hendrix.myopenid.com/ 是自己注册的 OpenID provider 。


还可以通过 openid-login.user-service-ref 指定 UserDetailsService 。


注意 user 元素中没有 password 属性，因为不需要。在 Spring Security 内部会随机生成一个密码，以防止出错。


### Attribute Exchange
可以从 OpenID provider 取得一些属性，例如：
```xml
  <openid-login>
    <attribute-exchange>
      <openid-attribute name="email" type="http://axschema.org/contact/email" required="true" />
      <openid-attribute name="name" type="http://axschema.org/namePerson" />
    </attribute-exchange>
  </openid-login>
```


openid-attribute 的属性：
- name - 依赖具体的 OpenID provider
- type - 是 URI ，依赖具体的 OpenID provider
- required - 如果为 true 表示只有成功取得这个属性才算 authentication 成功。


得到的 openid-attribute 属性值可以如下使用：
```java
        OpenIDAuthenticationToken token = (OpenIDAuthenticationToken) SecurityContextHolder.getContext()
                .getAuthentication();
        List<OpenIDAttribute> attributes = token.getAttributes();
```


可以有多个 attribute-exchange 元素，每个有不同的 identifier-matcher 属性。 identifier-matcher 属性是个正则表达式，与 user 的 OpenID identifier 匹配。参见  OpenID sample 。


FIXME 例子


## Response Headers
## Adding in Your Own Filters
可以
- 添加自己的过滤器
- 使用默认不在 namespace configuration 中的 Spring Security 过滤器，比如 CAS
- 定制标准的 standard namespace filter(例如定制 UsernamePasswordAuthenticationFilter) ，使用不同于默认值的设置


filters 的别名和顺序参见 org.springframework.security.config.http.SecurityFilters, 以及 HttpSecurityBeanDefinitionParser 第 166 行。


在 3.0 之前的版本里，排序是在 filter 初始化之后进行的，从 3.0+ 之后，排序在 filter 初始化之前进行。


文档中的过滤器表比 org.springframework.security.config.http.SecurityFilters 中少
- WEB_ASYNC_MANAGER_FILTER - 在 CONCURRENT_SESSION_FILTER 之后
- CORS_FILTER - 在 HEADERS_FILTER 之后
- OPENID_FILTER - 在 FORM_LOGIN_FILTER 之后
- LOGIN_PAGE_FILTER - 在 OPENID_FILTER 之后
- DIGEST_AUTH_FILTER - 在 LOGIN_PAGE_FILTER 之后， BASIC_AUTH_FILTER 之前
- REQUEST_CACHE_FILTER - 在 BASIC_AUTH_FILTER 之后


添加自定义的 filter ：
```xml
  <http>
    <custom-filter position="FORM_LOGIN_FILTER" ref="myFilter" />
  </http>

  <beans:bean id="myFilter" class="com.mycompany.MySpecialAuthenticationFilter" />
```


除 position 外，还可以使用 before/after 。使用 position 时值还可以是 FIRST/LAST 。


当要替换(position=它)某个标准 filter 时，注意还要将创建这个 filter 的元素（上表的最后一列）去掉，否则会出错。


SecurityContextPersistenceFilter, ExceptionTranslationFilter, FilterSecurityInterceptor 由 http 元素创建，因此无法被替换。


缺省情况下， http 元素还会自动创建 AnonymousAuthenticationFilter 和 SessionManagementFilter ，这两个 filter 可以被 disable
- AnonymousAuthenticationFilter - 使用 anonymous.enabled="false"
- SessionManagementFilter - 使用 session-management.session-fixation-protection="none"


如果被替换的 filter 需要有 authentication entry point ，则还需要添加 entry point bean(AuthenticationEntryPoint) ，并设置 &lt;http> 元素的 entry-point-ref 属性。可以参考 CAS 的例子。


# Method Security
从 2.0 开始， Spring Security 支持方法级别的安全，支持 JSR-250 和 Spring Security 自己的 @Secured 。从 3.0 开始，还可以使用新的基于表达式的注解。


有三种方式
- JSR-250 annotation
- Spring Security @Secured annotation
- new expression-based annotations(from v3.0)


注意： method-security 只会在与启用 method-security 在同一个 application context 的 bean 的方法上起作用。如果要 secure 非 spring bean 的对象的方法，需要使用 AspectJ


可以对单个 bean 应用安全：
```xml
<beans:bean>
  <intercept-methods>
    <protect access="" method="" />
  </intercept-methods>
</beans:bean>
```
也可以对很多 bean 应用安全（使用 AspectJ 切入点）


## The global-method-security Element
只能有一个 global-method-security 元素，用于
- enable annotation-based security
- security pointcut declarations


启用 @Secured
```xml
<global-method-security secured-annotations="enabled" />
```


在类或者接口的方法上添加注解
```java
    @Secured("IS_AUTHENTICATED_ANONYMOUSLY")
    public Account readAccount(Long id);
```


启用 JSR-250 annotations
```xml
  <global-method-security jsr250-annotations="enabled" />
```
允许简单的基于 role 的约束，但没有 @Secured 功能强。


启用 expression-based annotations
```xml
  <global-method-security pre-post-annotations="enabled" />
```


在类或者接口的方法上添加注解
```java
    @PreAuthorize("isAnonymous()")
    public Account[] findAccounts();

    @PreAuthorize("hasAuthority('ROLE_TELLER')")
    public Account post(Account account, double amount);
```


可以同时启用 @Secured, JSR-250, expression-based 注解，但在某个特定的 interface or class 上只能使用一种注解，如果在一个 interface or class 上使用了多个注解，只有一个会起作用。


AspectJ 的例子：
```xml
  <global-method-security>
    <protect-pointcut expression="execution(* com.mycompany.*Service.*(..))" access="ROLE_USER" />
  </global-method-security>
```


如果有多个 protect-pointcut ，则依次轮询，使用第一个符合条件的配置，就像 intercept-url 一样。


注解优先于 protect-pointcut 。


# The Default AccessDecisionManager
当使用命名空间时，会自动注册一个 AccessDecisionManager 实例，用于 method invocations and web URL access 访问控制，基于 intercept-url.access, protect-pointcut.access 以及 method 上的注解。


缺省的 AccessDecisionManager 是 AffirmativeBased ，使用 RoleVoter 和 AuthenticatedVoter 。


## Customizing the AccessDecisionManager
自定义 AccessDecisionManager
- 对 method security ，设置 global-method-security.access-decision-manager-ref
- 对 web url ，设置 http.access-decision-manager-ref
为 AccessDecisionManager bean id


# The Authentication Manager and the Namespace
使用 authentication-manager 元素会注册一个 ProviderManager 的实例作为 AuthenticationManager 。注意，通过命名空间使用 HTTP or method security 时不能自定义 AuthenticationManager ，但是可以自定义 AuthenticationProvider 。


例子
```xml
  <authentication-manager>
    <authentication-provider ref="casAuthenticationProvider" />
  </authentication-manager>

  <bean id="casAuthenticationProvider" class="org.springframework.security.cas.authentication.CasAuthenticationProvider">
  </bean>
```


可以为 AuthenticationManager 注册一个别名，以便在 application context 的其它地方用到：
```xml
  <security:authentication-manager alias="authenticationManager">
  </security:authentication-manager>

  <bean id="customizedFormLoginFilter" class="com.somecompany.security.web.CustomFormLoginFilter">
    <property name="authenticationManager" ref="authenticationManager" />
  </bean>
```
