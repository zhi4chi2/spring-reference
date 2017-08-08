# Runtime Environment
Spring Security 3.0 需要 JDK 5.0+


Spring Security 不需要修改 JRE 的配置文件，不需要特定 JAAS policy 配置，不需要在公用 classpath 下放置任何 jar 。在不同 EJB Container or Servlet Container 之间移动时不需要做额外操作，只拷贝 war/jar/ear 即可。


# Core Components
在 spring-security-core 中的 Java types


## SecurityContextHolder, SecurityContext and Authentication Objects
SecurityContextHolder 用于存储 SecurityContext 。


SecurityContextHolder 存储信息的三种方式：
- SecurityContextHolder.MODE_THREADLOCAL - 缺省，使用 ThreadLocal 。同一线程使用同一个 security identity.
- SecurityContextHolder.MODE_INHERITABLETHREADLOCAL - secure thread 及其子线程(spawn thread)使用同一个 security identity.
- SecurityContextHolder.MODE_GLOBAL - 对独立的(standalone) application, 同一 JVM 中所有线程使用同一个 security context


改变 SecurityContextHolder 存储信息方式的办法：
- 设置系统属性 "spring.security.strategy"(SecurityContextHolder.SYSTEM_PROPERTY)
- 调用静态方法 SecurityContextHolder.setStrategyName
可以设置为 "MODE_THREADLOCAL" | "MODE_INHERITABLETHREADLOCAL" | "MODE_GLOBAL" | SecurityContextHolderStrategy 的实现类完整类名


### Obtaining information about the current user
获得当前用户的信息：
```java
        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

        if (principal instanceof UserDetails) {
            String username = ((UserDetails) principal).getUsername();
        } else {
            String username = principal.toString();
        }
```


## The UserDetailsService
UserDetailsService 的实现有：
- InMemoryUserDetailsManager
- JdbcUserDetailsManager


## GrantedAuthority
Authentication 中还有 GrantedAuthority 信息，也是从 UserDetails 取得的。


Usually the GrantedAuthority objects are application-wide permissions. They are not specific to a given domain object.


## Summary
- SecurityContextHolder - 用于存储 SecurityContext 。
- SecurityContext - 持有 Authentication 以及可能的某个 request 特有的安全信息
- Authentication - 扩展 Principal ，表示 Spring Security 特有的 Principal 。当认证成功时，用 UserDetails 构造 Authentication
- UserDetails - 记录用户详细信息，特定于实际 application 。相当于实际用户表和 Spring Security 需要的 User 类之间的适配器。所以 application 需要实现 UserDetails 接口，从 application 自己的 User 表中取数据，构造 UserDetails 实现类对象，传给 Spring Security 。取 Principal 的时候再转成(cast) application 自己的 UserDetails 实现类对象，以便调用 application 特定的方法，比如 getEmail() 等等。
- GrantedAuthority - application 范围的权限集（比如角色），不包括 domain object security ，因为如果包括 domain object security 就太多了，认证用户时会很慢，还会使服务器 out of memory


# Authentication
## What is authentication in Spring Security
标准认证场景：
1. 提示用户输入用户名、密码
1. 检查用户名、密码正确
1. 取出用户的相关信息（比如角色）
1. 建立用户的 security context
1. 用户继续操作，由 access control mechanism 检查用户的 security context 中是否有足够的权限


Spring Security 的认证：
1. 用户名、密码被放入 UsernamePasswordAuthenticationToken 实例（ UsernamePasswordAuthenticationToken 实现 Authentication 接口）
1. UsernamePasswordAuthenticationToken 传入 AuthenticationManager 以认证
1. 如果认证成功， AuthenticationManager 返回一个 fully populated Authentication 实例
1. 调用 `SecurityContextHolder.getContext().setAuthentication(...)`


伪代码如下：
```java
        Authentication request = new UsernamePasswordAuthenticationToken(name, password);
        AuthenticationManager am = new SampleAuthenticationManager();
        Authentication result = am.authenticate(request);
        SecurityContextHolder.getContext().setAuthentication(result);
```


A user is authenticated when the SecurityContextHolder contains a fully populated Authentication object.


## Setting the SecurityContextHolder Contents Directly
Spring Security 并不关心怎么把 Authentication 放入 SecurityContextHolder ，它只要求在 AbstractSecurityInterceptor 检查用户权限之前， SecurityContextHolder 中已有 fully populated Authentication 。


因此，可以自定义 filters 或者 controllers 设置 Authentication(`SecurityContextHolder.getContext().setAuthentication(result)`) 以实现自定义的验证过程。从某个地方做认证，然后构建 Authentication 对象，放进 SecurityContextHolder 中。  In this case you also need to think about things which are normally taken care of automatically by the built-in authentication infrastructure. For example, you might need to pre-emptively create an HTTP session to cache the context between requests, before you write the response to the client


# Authentication in a Web Application
典型 Web 应用的验证过程：
1. 点链接
1. 服务器认为你请求了一个受保护的资源
1. 服务器返回一个 response 要求你先验证， response 可能是 HTTP response code ，或者重定向到登录页面
1. 根据验证方式的不同，浏览器呈现验证方式（如果是 form-login ，呈现一个登录页面；如果是 http-basic ，弹出一个对话框；还有 cookie, X.509 等等方式）
1. 浏览器发送验证数据给服务器。如果是 form-login ，则是 post 提交，如果是 http-basic 则是 HTTP header 信息。
1. 如果验证成功，转下一步，如果验证不成功，服务器通常会要求你再次认证，转第 2 步。
1. 如果有足够的权限访问第 1 步中的链接，请求成功，否则，会收到 403(forbidden) 错误。


Spring Security 中负责上述过程的类有：
- ExceptionTranslationFilter
- AuthenticationEntryPoint - 每种 authentication system 都有自己的 AuthenticationEntryPoint 实现
- authentication mechanism （负责调用 AuthenticationManager ）


- ExceptionTranslationFilter - 捕捉 Spring Security 产生的异常（一般由 AbstractSecurityInterceptor 抛出），如果用户没有认证，运行 AuthenticationEntryPoint ，如果已经认证但没有足够的权限，返回 403 代码。
- AuthenticationEntryPoint - 负责验证过程中的第 3 步，每个 authentication system 有自己的 AuthenticationEntryPoint 实现
- Authentication Mechanism - 例如 web form-login processing filter 。收集认证信息（用户名、密码），创建 request Authentication 对象，传入 AuthenticationManager ，如果认证成功得到返回的 fully-populated Authentication 对象，放入 SecurityContextHolder ，转向原来请求的 URL ；如果认证失败，转向重新认证。


## Storing the SecurityContext between requests
在 Web 应用中，需要在 requests 之间保存 SecurityContext ，保存在 session 中。


SecurityContextPersistenceFilter 负责在 requests 之间保存 SecurityContext ，把 SecurityContext 作为 Session 的一个属性保存。对每个 request 恢复 SecurityContext 并且在每个 request 结束后清理 SecurityContextHolder 。一般不应该从 session 中取 SecurityContext ，而是应该从 SecurityContextHolder 中取。


stateless RESTful web service 不能使用 session ，而是每个 request 都重新认证。但是仍然应该使用 SecurityContextPersistenceFilter ，因为它会在每个 request 结束后清理 SecurityContextHolder 。


缺省情况下， Session 中的 SecurityContext 和各个 request 的 thread 中的 SecurityContext 是同一个对象，因此，修改(SecurityContextHolder.getContext().setAuthentication(anAuthentication))任何一个都会影响所有线程的 SecurityContext 。
可以配置 SecurityContextPersistenceFilter 对每个 request 都创建新的 SecurityContext 。
还可以在临时修改 SecurityContext 之前用 `SecurityContextHolder.createEmptyContext()` 创建新的 context (SecurityContextHolder.setContext(SecurityContextHolder.createEmptyContext()))。


# Access-Control (Authorization) in Spring Security
AccessDecisionManager 有个方法：
- decide(Authentication authentication, Object object, Collection&lt;ConfigAttribute> configAttributes)


## Security and AOP Advice
Spring Security 使用 AOP 实现 method invocations 的 around advice ，使用 Filter 实现 web requests 的 around advice 。


## Secure Objects and the AbstractSecurityInterceptor
- secure object - 被保护的，可被授权的任何对象。最常见的例子是 web requests(org.springframework.security.web.FilterInvocation) 和 method invocations(org.aopalliance.intercept.MethodInvocation) 。每种 secure object 类型都有自己的 interceptor 类（扩展 AbstractSecurityInterceptor ）。
- security metadata attributes/configuration attributes(org.springframework.security.access.ConfigAttribute) - 作用于 secure object ，例如角色列表。其含义由 AccessDecisionManager 解释。 AbstractSecurityInterceptor 配置了 SecurityMetadataSource 来寻找 secure object 的 attributes 。 configuration attributes 配置在 secured methods 的 annotations 上，或者 secured URLs 的 access 属性上。
- SecurityMetadataSource - 用于寻找 secure object 的 attributes 。


AbstractSecurityInterceptor 定义了处理 secure object 请求的一致的流程：
1. 查找当前 request 关联的 "configuration attributes"
1. 调用 AccessDecisionManager 以决定是否有权限，传入参数 secure object, Authentication, configuration attributes
1. 可能会改变 Authentication 对象（ RunAsManager ）
1. 如果有权限，允许 secure object 继续执行
1. 如果 secure object 执行完毕，并且没有异常，如果配置了 AfterInvocationManager ，执行 AfterInvocationManager 。如果有异常，不会执行 AfterInvocationManager 。


AbstractSecurityInterceptor 中有个 RunAsManager 。 RunAsManager 会在授权之后（参见源码 AbstractSecurityInterceptor 第 233 行做授权，第 251 行调用 runAsManager.buildRunAs ）改变 SecurityContext 中的 Authentication 对象。这一般用于 call a remote system and present a different identity. Because Spring Security automatically propagates security identity from one server to another (assuming you’re using a properly-configured RMI or HttpInvoker remoting protocol client), this may be useful.


AfterInvocationManager 可以在 secure object invocation 成功执行之后改变 return object 或者甚至抛出异常。


AbstractSecurityInterceptor 依赖
- AuthenticationManager
- AccessDecisionManager
- SecurityMetadataSource - AbstractSecurityInterceptor.obtainSecurityMetadataSource()
- RunAsManager
- AfterInvocationManager


AbstractSecurityInterceptor 子类
- FilterSecurityInterceptor - 保护 FilterInvocation
- MethodSecurityInterceptor - 保护 MethodInvocation
  - AspectJMethodSecurityInterceptor - 保护 MethodInvocationAdapter(JoinPoint)


### Extending the Secure Object Model
例如创建新的 secure object 用以保护 messaging system 调用。


凡需要被保护，并且提供一种拦截调用的方式，都可以当做 secure object 。 Anything that requires security and also provides a way of intercepting a call (like the AOP around advice semantics) is capable of being made into a secure object


大多数应用只需要使用内建的三个 secure object
- org.springframework.security.web.FilterInvocation
- org.aopalliance.intercept.MethodInvocation
- org.aspectj.lang.JoinPoint


# Localization
国际化文件配置在 spring-security-core-4.2.3.RELEASE.jar!/org/springframework/security/messages.properties 。


需要如下配置：
```xml
  <bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basename" value="classpath:org/springframework/security/messages" />
  </bean>
```
但是其中只有简体中文版（ messages_zh_CN.properties ），没有繁体中文版。如果需要定义更多的语言，或者自定义消息内容，可以把文件拷贝到某个路径再修改，然后更改上面的配置即可。


注意，国际化需要 request 中已有 locale 信息（存储在 org.springframework.context.i18n.LocaleContextHolder 中）。虽然 DispatcherServlet 中会自动存储 locale 信息，但是它在 Spring Security 的 filters 之后运行。可以使用 RequestContextFilter 在 Spring Security 的 filters 之前存储 locale 信息。参考 contacts sample
