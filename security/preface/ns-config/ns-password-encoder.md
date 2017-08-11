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

  <beans:bean name="bcryptEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />

  <authentication-manager>
    <authentication-provider>
      <password-encoder ref="bcryptEncoder" />
      <user-service>
        <user name="jimi" password="$2a$10$oskZ3mzbgfktp4RLblclmeVIz7JsHRbfGBFxWo7ObuijoyZe/oY.i" authorities="ROLE_USER, ROLE_ADMIN" />
        <user name="bob" password="$2a$10$0I4paANLFtGNedZNbpiGe.UhGFsVC1Frh9elwf9ESYVVCMyNpKwSG" authorities="ROLE_USER" />
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
- 注销
- 输入错误密码，提示 Bad credentials


spring-security.xml 中加密的值可以如下得到
```java
package org.example.demo.spring.security;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

public class Test {
    public static void main(String[] args) {
        PasswordEncoder encoder = new BCryptPasswordEncoder();
        // $2a$10$22ZDFwd6AemxiPTFp2CRt.TCEipeG3OQnFua/XXcp8xIOx01Dl4cu
        System.out.println(encoder.encode("jimispassword"));
        // $2a$10$MXKsuONToO/97.ZpuLyAmO/FLRLolSYF2AM43QIGNeqkiWUcVNpey
        System.out.println(encoder.encode("bobspassword"));
    }
}
```


注意每次运行结果都不相同，即同一个 raw password 对应多个不同的加密值。

