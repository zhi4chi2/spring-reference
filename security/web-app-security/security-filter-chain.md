# DelegatingFilterProxy
DelegatingFilterProxy 连接 web.xml 和 application context 。


```xml
  <filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```


application context 中需要有个 bean id = filter-name 并且实现 Filter 接口（即 FilterChainProxy ）。


# FilterChainProxy
过滤器之间有依赖关系，因此它们的顺序非常重要。


FilterChainProxy 将多个 filter 组合在一起，使得 web.xml 不会杂乱。
```xml
<bean id="filterChainProxy" class="org.springframework.security.web.FilterChainProxy">
  <constructor-arg>
    <list>
      <sec:filter-chain pattern="/restful/**"
        filters="
           securityContextPersistenceFilterWithASCFalse,
           basicAuthenticationFilter,
           exceptionTranslationFilter,
           filterSecurityInterceptor" />
      <sec:filter-chain pattern="/**"
        filters="
           securityContextPersistenceFilterWithASCTrue,
           formLoginFilter,
           exceptionTranslationFilter,
           filterSecurityInterceptor" />
    </list>
  </constructor-arg>
</bean>
```


sec:filter-chain 对应 SecurityFilterChain ， filters 属性中是过滤器的 bean 名字，各个过滤器调用的顺序就是 filters 属性中出现的顺序。


注意： FilterChainProxy 不会调用配置的 filters 的属于标准 Filter 的 lifecycle(init, destroy) 方法，可以使用 Spring 的 lifecycle(afterPropertiesSet, destroy) 方法代替。


## Bypassing the Filter Chain
可以使用 filters="none" 绕过过滤器链。 that anything matching this path will then have no authentication or authorization services applied, the SecurityContextHolder will not have been populated and the contents will be null.


参见 org.springframework.security.config.http.FilterChainBeanDefinitionParser


# Filter Ordering
# Request Matching and HttpFirewall
在 FilterChainProxy 匹配过滤器链(filter-chain/http.pattern)以及 FilterSecurityInterceptor 匹配 security constraints (intercept-url) 时，使用的是 requestURI 中的 servletPath, pathInfo 部分。


但是 servlet spec 没有明确指定 servletPath and pathInfo 可以包含的值，比如 RFC 2396 中规定 URL 的每个部分都可以包含 parameters, Specification 中没有指明 servletPath and pathInfo 中是否可以有 parameters, 在不同 servlet containers 中表现也不一样。黑客可以故意添加 parameters 以造成匹配成功或失败。还有其它可能，比如路径中含有 "/../" 或者 "//" 也会造成匹配失败。有些 servlet containers 会在匹配 servlet mapping 前 normalize URI ，但有些 servlet containers 则不会。


所以， FilterChainProxy 使用 HttpFirewall 拒绝 Un-normalized 的 URL ，并去掉 path parameters （包括 servletPath and pathInfo 中的 parameters ）和多余的 / 后再匹配 pattern 。


缺省会使用 AntPathRequestMatcher 匹配 pattern ，对 servletPath and pathInfo 的串做不区分大小写的匹配。


还可以使用 RegexRequestMatcher 匹配。


最佳实践：在最后使用 "/**" 或 "**" 匹配所有，并配置为拒绝访问（ deny access ）。


service 层方法上的 Security 配置更 robust 而且不容易被跳过，因此推荐使用。


# Use with other Filter-Based Frameworks
当和 SiteMesh/Wicket 一起使用时， Spring Security 的过滤器需要在最前面，以保证 SecurityContextHolder 的建立。


# Advanced Namespace Configuration
如果有多个 http 元素，每个 http 元素都在内部 FilterChainProxy bean 中创建一个 filter chain, http 元素的顺序很重要，会依次匹配。


例子
```xml
<http pattern="/restful/**" create-session="stateless">
  <intercept-url pattern='/**' access='hasRole('REMOTE')' />
  <http-basic />
</http>

<http pattern="/login.htm*" security="none" />

<http>
  <intercept-url pattern='/**' access='hasRole('USER')' />
  <form-login login-page='/login.htm' default-target-url="/home.htm" />
  <logout />
</http>
```
