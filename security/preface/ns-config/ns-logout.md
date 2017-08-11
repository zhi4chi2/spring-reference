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
    <input type="submit" value="Logoff" />
    <security:csrfInput />
  </form>
</body>
</html>
```


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 /
- 点 Logoff 注销，显示 http://localhost:8080/demo-spring-security/login.jsp?logout


注意，不能直接访问 http://localhost:8080/demo-spring-security/logout 注销，因为 logout 需要用 POST 方式提交，而不能是 GET 。


如果将 spring-security.xml 改为
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <http-basic />
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
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 /
- 点 Logoff 注销，提交 /logout 成功然后转向 http://localhost:8080/demo-spring-security/login?logout 但页面显示 404 。但在地址栏输入 http://localhost:8080/demo-spring-security/index.jsp 可以显示，证明并没有 logout ！可能 basic login 不能 logout 吧。 basic login 每个请求都在 request header 中有 `Authorization:Basic amltaTpqaW1pc3Bhc3N3b3Jk` 字样。

