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
