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
<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>Hello, world!
</body>
</html>
```


测试
- 访问 http://localhost:8080/demo-spring-security/ 输入密码，进入 index.jsp


# 不能与 form-login 一起使用
文档中说
> Basic authentication will then take precedence and will be used to prompt for a login when a user attempts to access a protected resource. Form login is still available in this configuration if you wish to use it, for example through a login form embedded in another web page.


但实测不是这样，如果有 form-login 则一定先使用 form-login 。参见 SecurityFilters 中 filter 的顺序。


更改 spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <http-basic />
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


测试
- 访问 localhost:8080/demo-spring-security/ 将重定向到 http://localhost:8080/demo-spring-security/login
- 输入用户名密码，进入 index.jsp
期间没有 basic login 什么事。


# 不能与 login.jsp 一起使用
如果不使用 form-login 而是另外有 login.jsp(代码参见 [Form Login Options](/security/preface/ns-config/ns-form.md))


在 spring-security.xml 中定义
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/login.jsp*" access="permitAll" />
    <intercept-url pattern="/login*" access="permitAll" />
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


测试
- 访问 http://localhost:8080/demo-spring-security/ 通过 basic 方式输入用户名密码，进入 index.jsp
- 关闭再重新打开浏览器
- 访问 http://localhost:8080/demo-spring-security/login.jsp 输入用户名密码，提交的 http://localhost:8080/demo-spring-security/login 报 404 错误。这个 url 的拦截是由 form-login(UsernamePasswordAuthenticationFilter) 做的。因为没有定义 form-login 所以报 404


同理 basic login 不能与 logout 一起使用。因为默认 logout 之后返回 http://localhost:8080/demo-spring-security/login?logout ，这个 url 也会报 404 错误。


可以将 index.jsp 改为 [Form Login Options](/security/preface/ns-config/ns-form.md) 中的样子测试 logout 。


如果 basic login 要与 logout 一起使用，应该需要自定义 logout-success-url
