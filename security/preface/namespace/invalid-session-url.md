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
    <session-management invalid-session-url="/invalidSession.html" />
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


src/main/webapp/invalidSession.html
```html
<!DOCTYPE html>
<html>
<head>
<title>Invalid Session</title>
</head>
<body>Invalid Session
</body>
</html>
```


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 /
- 点 Logoff 注销，再次登录，显示 http://localhost:8080/demo-spring-security/invalidSession.html


点 Logoff 后的流程是这样的
1. 首先 post /logout, JSESSIONID=2764188BD94656CDE36AEE740276E560 。返回 302 redirect /login.jsp?logout
2. GET 请求 /login.jsp?logout JSESSIONID=2764188BD94656CDE36AEE740276E560 。返回 302 redirect /invalidSession.html JSESSIONID=F673444C3A941FC6CCDEA147B0507DF8
3. GET 请求 /invalidSession.html JSESSIONID=F673444C3A941FC6CCDEA147B0507DF8 。返回 302 redirect /login.jsp
4. GET 请求 /login.jsp JSESSIONID=F673444C3A941FC6CCDEA147B0507DF8 ，返回 200
5. 输入用户名密码，提交。 POST 请求 /login JSESSIONID=F673444C3A941FC6CCDEA147B0507DF8 ，返回 302 redirect /invalidSession.html JSESSIONID=E08C02780C795AEA59AFA3DA2AD5F983
6. GET 请求 /invalidSession.html 返回 200


所以虽然与文档中说的结果一致，但实际原理是不一样的。


Note that if you use this mechanism to detect session timeouts, it may falsely report an error if the user logs out and then logs back in without closing the browser. This is because the session cookie is not cleared when you invalidate the session and will be resubmitted even if the user has logged out.


也因此 logout delete-cookies 是不起作用的。


修改 spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/login.jsp*" access="permitAll" />
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login login-page="/login.jsp" authentication-failure-url="/login.jsp?login_error=1" />
    <logout delete-cookies="JSESSIONID" />
    <session-management invalid-session-url="/invalidSession.html" />
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
- 点 Logoff 注销，再次登录，显示 http://localhost:8080/demo-spring-security/invalidSession.html


点 Logoff 后的流程是这样的
1. 首先 post /logout, JSESSIONID=A81704D5483A3C327C9922097FC0973D 。返回 302 redirect /login.jsp?logout Set-Cookie:JSESSIONID=""; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Path=/demo-spring-security
2. GET 请求 /login.jsp?logout JSESSIONID=A81704D5483A3C327C9922097FC0973D 。返回 302 redirect /invalidSession.html JSESSIONID=0017446216C7E5870954CABD4F4BF56C
3. GET 请求 /invalidSession.html JSESSIONID=0017446216C7E5870954CABD4F4BF56C 。返回 302 redirect /login.jsp
4. GET 请求 /login.jsp JSESSIONID=0017446216C7E5870954CABD4F4BF56C ，返回 200
5. 输入用户名密码，提交。 POST 请求 /login JSESSIONID=0017446216C7E5870954CABD4F4BF56C ，返回 302 redirect /invalidSession.html JSESSIONID=AE41F77B894114CCF10374F6319FE60A
6. GET 请求 /invalidSession.html 返回 200


与没有加 logout delete-cookies 的区别在
1. 第一步时返回的 response Set-Cookie:JSESSIONID=""


但这其实没有用，因为 redirect 是浏览器自动操作，第 2 步时依然使用原来的 JSESSIONID(A81704D5483A3C327C9922097FC0973D)


文档中说
> Unfortunately this can’t be guaranteed to work with every servlet container, so you will need to test it in your environment


但这其实并不是 servlet container 的原因，而是浏览器的，测试 Firefox/Chrome 都会出现这样的问题，因此 logout delete-cookies 其实无效。
