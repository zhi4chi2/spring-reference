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


# permitAll
src/main/resources/spring-security.xml
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


src/main/webapp/login.jsp
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ page pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<title>Login</title>
</head>

<body onload="document.f.username.focus();">
  <h1>Login</h1>

  <c:if test="${not empty param.login_error}">
    <font color="red"> Your login attempt was not successful, try again.<br /> <br /> Reason: <c:out value="${SPRING_SECURITY_LAST_EXCEPTION.message}" />.
    </font>
  </c:if>

  <form name="f" action="<c:url value='/login' />" method="POST">
    <table>
      <tr>
        <td>User:</td>
        <td><input type='text' name='username' /></td>
      </tr>
      <tr>
        <td>Password:</td>
        <td><input type='password' name='password'></td>
      </tr>

      <tr>
        <td colspan='2'><input name="submit" type="submit"></td>
      </tr>
      <tr>
        <td colspan='2'><input name="reset" type="reset"></td>
      </tr>
    </table>
    <input type="hidden" name="<c:out value="${_csrf.parameterName}"/>" value="<c:out value="${_csrf.token}"/>" />
  </form>
</body>
</html>
```


src/main/webapp/index.jsp
```jsp
<%@ page pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags"%>

<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>
  <form action="<c:url value='/logout' />" method="post">
    <input type="submit" value="Logoff" /> (also clears any remember-me cookie)
    <security:csrfInput />
  </form>
</body>
</html>
```


浏览器访问 http://localhost:8080/demo-spring-security/ 测试：
- 正确输入用户名密码，进入 index.jsp
- log out
- 错误输入用户名密码，提示出错


注意将 login.jsp 的第 21 行改为
```jsp
        <td><input type='text' name='username' value='<c:if test="${not empty param.login_error}"><c:out value="${SPRING_SECURITY_LAST_USERNAME}"/></c:if>' /></td>
```
并不会起作用，因为 UsernamePasswordAuthenticationFilter.SPRING_SECURITY_LAST_USERNAME_KEY 在 spring security 3 中已经标为 deprecated ，在 spring 4 中已经删除。如果想得到 last username 需要自定义 AuthenticationFailureHandler


# security="none" 方式
src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http pattern="/login.jsp*" security="none" />
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1" />
    <logout />
    <csrf disabled="true" />
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


注意第 10 行要定义 csrf disabled ，默认是启用 csrf 的。


同时 login.jsp 中要去掉（可选）第 35 行的
```jsp
    <input type="hidden" name="<c:out value="${_csrf.parameterName}"/>" value="<c:out value="${_csrf.token}"/>" />
```
否则因为 login.jsp 的 security="none" 使得实际生成
```html
    <input type="hidden" name="" value="" />
```
因此如果 spring-security.xml 中没有加第 10 行 csrf disabled 的话，就会导致 csrf 出错。


综上， security="none" 方式不如 access="permitAll" 方式。


# form-login 属性
If a form login isn't prompted by an attempt to access a protected resource, the default-target-url option comes into play. This is the URL the user will be taken to after successfully logging in, and defaults to "/". You can also configure things so that the user always ends up at this page (regardless of whether the login was "on-demand" or they explicitly chose to log in) by setting the always-use-default-target attribute to "true". This is useful if your application always requires that the user starts at a "home" page


在 [access="permitAll" 方式](#permitAll)的基础上，添加 src/main/webapp/hello.html
```html
<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>Hello, World!
</body>
</html>
```


然后修改 src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/login.jsp*" access="permitAll" />
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1"
      default-target-url='/hello.html' />
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


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 index.jsp
- 注销
- 再次登入，进入 hello.html
- 访问 http://localhost:8080/demo-spring-security/index.jsp 注销
- 访问 http://localhost:8080/demo-spring-security/login.jsp 登入，进入 hello.html


再次修改 src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/login.jsp*" access="permitAll" />
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1"
      default-target-url='/hello.html' always-use-default-target='true' />
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


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 hello.html
- 访问 http://localhost:8080/demo-spring-security/index.jsp 注销
- 访问 http://localhost:8080/demo-spring-security/login.jsp 登入，进入 hello.html
