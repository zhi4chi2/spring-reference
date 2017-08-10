# Overview
注意， Remember-me 需要 UserDetailsService ，即使 authentication provider 不使用 UserDetailsService ，比如 LDAP provider ，也要使用 UserDetailsService 来得到 Remember-me 功能。


# Simple Hash-Based Token Approach
在某次成功的交互式认证之后，发送到浏览器一个 cookie ，格式如下：
```
base64(username + ":" + expirationTime + ":" + md5Hex(username + ":" + expirationTime + ":" password + ":" + key))

username:          As identifiable to the UserDetailsService
password:          That matches the one in the retrieved UserDetails
expirationTime:    The date and time when the remember-me token expires, expressed in milliseconds
key:               A private key to prevent modification of the remember-me token
```
参见 `org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices.onLoginSuccess(HttpServletRequest, HttpServletResponse, Authentication)`


在 `org.springframework.security.web.authentication.rememberme.AbstractRememberMeServices.autoLogin(HttpServletRequest, HttpServletResponse)` 中会重新用 username, password, key 计算签名，与 cookie 中的签名比对，如果不一致，则认证失败。所以 remember me 需要密码没有修改过，如果修改密码，则 remember me 失效。


If a principal is aware a token has been captured, they can easily change their password and immediately invalidate all remember-me tokens on issue.


配置示例：
```xml
<remember-me key="myAppKey" />
```
自动从 application context 中查找 UserDetailsService ，如果有多个，则需要用 user-service-ref 属性指定。


# Persistent Token Approach
参考 http://jaspan.com/improved_persistent_login_cookie_best_practice


配置示例：
```xml
<remember-me data-source-ref="someDataSource" />
```


数据库中需要有表：
```sql
CREATE TABLE persistent_logins
  (
     username  VARCHAR(64) NOT NULL,
     series    VARCHAR(64) PRIMARY KEY,
     token     VARCHAR(64) NOT NULL,
     last_used TIMESTAMP NOT NULL
  );
```


# Remember-Me Interfaces and Implementations
Remember-me 通常与 UsernamePasswordAuthenticationFilter 一起使用，在 AbstractAuthenticationProcessingFilter 中有个 RememberMeServices 成员变量。


Remember-me 也可以与 basic 认证(BasicAuthenticationFilter)一起使用。


## TokenBasedRememberMeServices
RememberMeAuthenticationFilter.doFilter 调用 RememberMeServices.autoLogin 。


TokenBasedRememberMeServices 生成 RememberMeAuthenticationToken ，再由 RememberMeAuthenticationFilter.doFilter 调用 authenticationManager.authenticate 处理认证。实际由 RememberMeAuthenticationProvider 处理。


TokenBasedRememberMeServices 和 RememberMeAuthenticationProvider 需要使用同样的 key ，即 RememberMeAuthenticationToken.getKeyHash() == RememberMeAuthenticationProvider.getKey().hashCode() ，参见 RememberMeAuthenticationProvider.authenticate 方法。


TokenBasedRememberMeServices 还需要 UserDetailsService 以取得 GrantedAuthority 。


TokenBasedRememberMeServices 还实现 LogoutHandler 接口，可以和 LogoutFilter 一起使用，在登出时清除 cookie 。


配置示例
```xml
<bean id="rememberMeFilter" class="org.springframework.security.web.authentication.rememberme.RememberMeAuthenticationFilter">
  <property name="rememberMeServices" ref="rememberMeServices" />
  <property name="authenticationManager" ref="theAuthenticationManager" />
</bean>

<bean id="rememberMeServices" class="org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices">
  <property name="userDetailsService" ref="myUserDetailsService" />
  <property name="key" value="springRocks" />
</bean>

<bean id="rememberMeAuthenticationProvider" class="org.springframework.security.authentication.rememberme.RememberMeAuthenticationProvider">
  <property name="key" value="springRocks" />
</bean>
```


还要
- UsernamePasswordAuthenticationFilter.setRememberMeServices() 注入 RememberMeServices
- AuthenticationManager.setProviders() 注入 rememberMeAuthenticationProvider
- 添加 RememberMeAuthenticationFilter 到 FilterChainProxy 过滤器链


# PersistentTokenBasedRememberMeServices
使用方式与 TokenBasedRememberMeServices 一样，但是需要配置 PersistentTokenRepository 。


有两个内建 PersistentTokenRepository 实现
- InMemoryTokenRepositoryImpl
- JdbcTokenRepositoryImpl
