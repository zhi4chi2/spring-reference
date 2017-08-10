```java
package org.example.demo.spring.security;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-security.xml");
        String[] names = applicationContext.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name + " : " + applicationContext.getBean(name).getClass());
        }
        applicationContext.close();
    }
}
```


对如下的 src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/login.jsp*" access="permitAll" />
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1" />
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


输出
```
org.springframework.security.filterChains : class java.util.ArrayList
org.springframework.security.filterChainProxy : class org.springframework.security.web.FilterChainProxy
org.springframework.security.web.PortMapperImpl#0 : class org.springframework.security.web.PortMapperImpl
org.springframework.security.web.PortResolverImpl#0 : class org.springframework.security.web.PortResolverImpl
org.springframework.security.config.authentication.AuthenticationManagerFactoryBean#0 : class org.springframework.security.authentication.ProviderManager
org.springframework.security.authentication.ProviderManager#0 : class org.springframework.security.authentication.ProviderManager
org.springframework.security.web.csrf.LazyCsrfTokenRepository#0 : class org.springframework.security.web.csrf.LazyCsrfTokenRepository
org.springframework.security.web.context.HttpSessionSecurityContextRepository#0 : class org.springframework.security.web.context.HttpSessionSecurityContextRepository
org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy#0 : class org.springframework.security.web.authentication.session.CompositeSessionAuthenticationStrategy
org.springframework.security.web.savedrequest.HttpSessionRequestCache#0 : class org.springframework.security.web.savedrequest.HttpSessionRequestCache
org.springframework.security.config.http.HttpConfigurationBuilder$SecurityContextHolderAwareRequestFilterBeanFactory#0 : class org.springframework.security.config.http.HttpConfigurationBuilder$SecurityContextHolderAwareRequestFilterBeanFactory
org.springframework.security.config.http.FilterInvocationSecurityMetadataSourceParser$DefaultWebSecurityExpressionHandlerBeanFactory#0 : class org.springframework.security.config.http.FilterInvocationSecurityMetadataSourceParser$DefaultWebSecurityExpressionHandlerBeanFactory
org.springframework.security.config.http.FilterInvocationSecurityMetadataSourceParser$DefaultWebSecurityExpressionHandlerBeanFactory#0$created#0 : class org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler
org.springframework.security.access.vote.AffirmativeBased#0 : class org.springframework.security.access.vote.AffirmativeBased
org.springframework.security.web.access.intercept.FilterSecurityInterceptor#0 : class org.springframework.security.web.access.intercept.FilterSecurityInterceptor
org.springframework.security.web.access.DefaultWebInvocationPrivilegeEvaluator#0 : class org.springframework.security.web.access.DefaultWebInvocationPrivilegeEvaluator
org.springframework.security.authentication.AnonymousAuthenticationProvider#0 : class org.springframework.security.authentication.AnonymousAuthenticationProvider
org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter#0 : class org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
org.springframework.security.userDetailsServiceFactory : class org.springframework.security.config.http.UserDetailsServiceFactoryBean
org.springframework.security.web.DefaultSecurityFilterChain#0 : class org.springframework.security.web.DefaultSecurityFilterChain
org.springframework.security.provisioning.InMemoryUserDetailsManager#0 : class org.springframework.security.provisioning.InMemoryUserDetailsManager
org.springframework.security.authentication.dao.DaoAuthenticationProvider#0 : class org.springframework.security.authentication.dao.DaoAuthenticationProvider
org.springframework.security.authentication.DefaultAuthenticationEventPublisher#0 : class org.springframework.security.authentication.DefaultAuthenticationEventPublisher
org.springframework.security.authenticationManager : class org.springframework.security.authentication.ProviderManager
```
