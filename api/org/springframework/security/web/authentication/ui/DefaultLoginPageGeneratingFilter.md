如果没有显式定义 form-login.login-page 属性，则用 DefaultLoginPageGeneratingFilter 生成登录页面。


生成的登录页面大致如下：
```html
<html>
<head>
<title>Login Page</title>
</head>
<body onload='document.f.username.focus();'>
  <h3>Login with Username and Password</h3>
  <form name='f' action='/demo-spring-security/login' method='POST'>
    <table>
      <tr>
        <td>User:</td>
        <td><input type='text' name='username' value=''></td>
      </tr>
      <tr>
        <td>Password:</td>
        <td><input type='password' name='password' /></td>
      </tr>
      <tr>
        <td colspan='2'><input name="submit" type="submit" value="Login" /></td>
      </tr>
      <input name="_csrf" type="hidden" value="4a6d3b25-e422-4752-af28-14b815197c2f" />
    </table>
  </form>
</body>
</html>
```


