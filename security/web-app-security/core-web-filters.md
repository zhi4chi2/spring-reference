# FilterSecurityInterceptor
FilterSecurityInterceptor 配置示例：
```xml
<bean id="filterSecurityInterceptor" class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
  <property name="authenticationManager" ref="authenticationManager" />
  <property name="accessDecisionManager" ref="accessDecisionManager" />
  <property name="securityMetadataSource">
    <security:filter-security-metadata-source>
      <security:intercept-url pattern="/secure/super/**" access="ROLE_WE_DONT_HAVE" />
      <security:intercept-url pattern="/secure/**" access="ROLE_SUPERVISOR,ROLE_TELLER" />
    </security:filter-security-metadata-source>
  </property>
</bean>
```


SecurityMetadataSource 用于对一个 url 返回 `Collection<ConfigAttribute>`


或使用正则表达式形式：
```xml
<bean id="filterInvocationInterceptor" class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
  <property name="authenticationManager" ref="authenticationManager" />
  <property name="accessDecisionManager" ref="accessDecisionManager" />
  <property name="runAsManager" ref="runAsManager" />
  <property name="securityMetadataSource">
    <security:filter-security-metadata-source path-type="regex">
      <security:intercept-url pattern="\A/secure/super/.*\Z" access="ROLE_WE_DONT_HAVE" />
      <security:intercept-url pattern="\A/secure/.*\" access="ROLE_SUPERVISOR,ROLE_TELLER" />
    </security:filter-security-metadata-source>
  </property>
</bean>
```


# ExceptionTranslationFilter
ExceptionTranslationFilter 处理 FilterSecurityInterceptor 抛出的异常。


ExceptionTranslationFilter 配置示例：
```xml
<bean id="exceptionTranslationFilter" class="org.springframework.security.web.access.ExceptionTranslationFilter">
  <property name="authenticationEntryPoint" ref="authenticationEntryPoint" />
  <property name="accessDeniedHandler" ref="accessDeniedHandler" />
</bean>

<bean id="authenticationEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
  <property name="loginFormUrl" value="/login.jsp" />
</bean>

<bean id="accessDeniedHandler" class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
  <property name="errorPage" value="/accessDenied.htm" />
</bean>
```


## AuthenticationEntryPoint
实际使用哪个 AuthenticationEntryPoint 依赖于 authentication mechanism 。


## AccessDeniedHandler
AccessDeniedHandler.errorPage 可以是一个页面，也可以是 MVC controller 。默认的 accessDeniedHandler 是 AccessDeniedHandlerImpl ，参见 ExceptionTranslationFilter 第 79 行。


## SavedRequest s and the RequestCache Interface
ExceptionTranslationFilter 还会在调用 AuthenticationEntryPoint 之前保存 current request 以在用户认证之后恢复 request 。比如 form-login 登入后使用 SavedRequestAwareAuthenticationSuccessHandler 转向原来的 URL 。


RequestCache 封装了取得和存储 HttpServletRequest 的功能。默认实现类是 HttpSessionRequestCache ，使用 HttpSession 存储 request 。


当用户被重定向到原来的 URL 后， RequestCacheAwareFilter 从 cache 中恢复 request 。


# SecurityContextPersistenceFilter
简单配置示例
```xml
<bean id="securityContextPersistenceFilter" class="org.springframework.security.web.context.SecurityContextPersistenceFilter" />
```


## SecurityContextRepository
从 Spring Security 3.0 开始，加载和存储 security context 的功能封装在接口 SecurityContextRepository 中。默认实现是 HttpSessionSecurityContextRepository （参见 SecurityContextPersistenceFilter 第 68 行），将 SecurityContext 保存在 HttpSession attribute "SPRING_SECURITY_CONTEXT" 中。


# UsernamePasswordAuthenticationFilter
简单配置示例
```xml
<bean id="authenticationFilter" class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
  <property name="authenticationManager" ref="authenticationManager" />
</bean>
```


## Application Flow on Authentication Success and Failure
UsernamePasswordAuthenticationFilter 仅在 URL 匹配 filterProcessesUrl （默认是 "/login" ）时才做实际动作。验证通过后调用 AuthenticationSuccessHandler ，失败调用 AuthenticationFailureHandler 。


AuthenticationSuccessHandler 子类有：
- ForwardAuthenticationSuccessHandler
- SimpleUrlAuthenticationSuccessHandler
  - SavedRequestAwareAuthenticationSuccessHandler - 重定向到认证前的 URL


AuthenticationFailureHandler 子类有：
- DelegatingAuthenticationFailureHandler
- ForwardAuthenticationFailureHandler
- SimpleUrlAuthenticationFailureHandler
  - ExceptionMappingAuthenticationFailureHandler

