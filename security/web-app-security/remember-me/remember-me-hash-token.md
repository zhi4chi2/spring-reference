src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
    <logout />
    <remember-me />
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
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags"%>

<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>
  <p>
    Your principal object is:
    <%=request.getUserPrincipal()%>
  </p>
  <form action="<c:url value='/logout' />" method="post">
    <input type="submit" value="Logoff" />
    <security:csrfInput />
  </form>
</body>
</html>
```


测试
- 访问 localhost:8080/demo-spring-security/ 不勾选 Remember me on this computer 登录
- 关闭浏览器，再次访问 localhost:8080/demo-spring-security/ 还是需要登录，此次勾选 Remember me on this computer
- 再次关闭浏览器，再次访问 http://localhost:8080/demo-spring-security/ 不需要再登录，显示 RememberMeAuthenticationToken
- 点 Logoff 注销
- 关闭浏览器，再次访问 localhost:8080/demo-spring-security/ 需要登录
- 重启 tomcat ，再次访问 localhost:8080/demo-spring-security/ 需要登录(Set-Cookie:remember-me=""; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Path=/demo-spring-security)


当勾选 Remember me on this computer 时， /login 的 response 会加上 cookie
```
remember-me=amltaToxNTAzNjM5ODU0OTgwOjE5Yjc1N2EyYTNkZDNiOTY5ZTE2ZjQ2NjBlMzExNzEz; Expires=Fri, 25-Aug-2017 05:44:14 GMT; Path=/demo-spring-security;
```


当关闭浏览器再次访问时，会将此 cookie 发给服务器。


所以， TokenBasedRememberMeServices 需要
- tomcat 没有重启
- 没有注销
