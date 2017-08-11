src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
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


src/main/webapp/index.jsp
```jsp
<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>Hello world!
</body>
</html>
```


测试
- 访问 http://localhost:8080/demo-spring-security/ 输入一个不存在的用户，显示 Bad credentials


Bad credentials 定义在 AbstractUserDetailsAuthenticationProvider 第 151 行。


之所以没有显示成中文是因为第 87 行定义的 `protected MessageSourceAccessor messages = SpringSecurityMessageSource.getAccessor();` 被 `MessageSourceAware` 接口的 `setMessageSource(MessageSource messageSource)` 覆盖了。而 web.xml 中定义的 XmlWebApplicationContext 中没有定义 messageSource 。所以 `messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials")` 实际会得到 NoSuchMessageException 的，所以最终使用的是 defaultMessage(Bad credentials)


# messageSource
更改 spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
  </http>

  <authentication-manager>
    <authentication-provider>
      <user-service>
        <user name="jimi" password="jimispassword" authorities="ROLE_USER, ROLE_ADMIN" />
        <user name="bob" password="bobspassword" authorities="ROLE_USER" />
      </user-service>
    </authentication-provider>
  </authentication-manager>

  <beans:bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <beans:property name="basename" value="classpath:i18n/messages" />
  </beans:bean>
</beans:beans>
```


增加 src/main/resources/i18n/messages\_en\_US.properties
```
AbstractUserDetailsAuthenticationProvider.badCredentials=oh, bad
```


增加 src/main/resources/i18n/messages\_zh\_CN.properties
```
AbstractUserDetailsAuthenticationProvider.badCredentials=\u574F\u4E86
```


测试
- 访问 http://localhost:8080/demo-spring-security/ 输入一个不存在的用户，显示【坏了】
- 在浏览器中更改语言为英文，使得 `Accept-Language: en-US,zh-CN` ，再次访问 http://localhost:8080/demo-spring-security/ 输入一个不存在的用户，仍然显示【坏了】


# RequestContextFilter
更改 web.xml
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID"
  version="2.5">
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring-*.xml</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <filter>
    <filter-name>localizationFilter</filter-name>
    <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>localizationFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

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


测试
- 在浏览器中更改语言为英文，使得 `Accept-Language: en-US,zh-CN` ，再次访问 http://localhost:8080/demo-spring-security/ 输入一个不存在的用户，显示 oh, bad
- 改回浏览器语言为中文，使得 `Accept-Language: zh-CN,zh;q=0.8,en-US;` ，再次访问 http://localhost:8080/demo-spring-security/ 输入一个不存在的用户，显示【坏了】

