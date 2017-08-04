src/main/webapp/WEB-INF/web.xml
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
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```


src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <http-basic />
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
<%@ page pageEncoding="UTF-8"%>

<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>Hello, world!
</body>
</html>
```


访问 http://localhost:8080/demo-spring-security/


文档中说
> Basic authentication will then take precedence and will be used to prompt for a login when a user attempts to access a protected resource. Form login is still available in this configuration if you wish to use it, for example through a login form embedded in another web page.


但实测不是这样，即使有 login.jsp 并且在 spring-security.xml 中定义
```xml
    <intercept-url pattern="/login.jsp*" access="permitAll" />
```
访问 http://localhost:8080/demo-spring-security/login.jsp 在提交时浏览器还是会弹出对话框让输入用户名密码，相当于输入两遍用户名密码。而且提交的 http://localhost:8080/demo-spring-security/login 报 404 错误。这个 url 的拦截应该是由 form-login(UsernamePasswordAuthenticationFilter) 做的。


如果 basic login 与 logout 一起使用：默认 logout 之后返回 http://localhost:8080/demo-spring-security/login?logout ，这个 url 也会报 404 错误。
