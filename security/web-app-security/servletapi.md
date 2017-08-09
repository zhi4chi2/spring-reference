参见 spring-security-4.2.3.RELEASE\samples\xml\servletapi sample


# Servlet 2.5+ Integration
## HttpServletRequest.getRemoteUser()
HttpServletRequest.getRemoteUser() 返回 `SecurityContextHolder.getContext().getAuthentication().getName()` ，一般是 username 。如果为 null 表示是匿名用户。


## HttpServletRequest.getUserPrincipal()
HttpServletRequest.getUserPrincipal() 返回 `SecurityContextHolder.getContext().getAuthentication()` 。


例子
```java
        Authentication auth = (Authentication) httpServletRequest.getUserPrincipal();
        MyCustomUserDetails userDetails = (MyCustomUserDetails) auth.getPrincipal();
        String firstName = userDetails.getFirstName();
        String lastName = userDetails.getLastName();
```


## HttpServletRequest.isUserInRole(String)
HttpServletRequest.isUserInRole(String) 检查 `SecurityContextHolder.getContext().getAuthentication().getAuthorities()` 中是否包含指定的角色。


isUserInRole 的参数不要带 "ROLE_" 前缀，会自动加上。


例子
```java
boolean isAdmin = httpServletRequest.isUserInRole("ADMIN");
```


# Servlet 3+ Integration
## HttpServletRequest.authenticate(HttpServletRequest,HttpServletResponse)
HttpServletRequest.authenticate(HttpServletRequest,HttpServletResponse) 用于保证用户已经认证，如果没有登录，使用配置的 AuthenticationEntryPoint 使用户做登入认证。


## HttpServletRequest.login(String,String)
HttpServletRequest.login(String,String) 使用当前的 AuthenticationManager 做认证。


例子
```java
        try {
            httpServletRequest.login("user", "password");
        } catch (ServletException e) {
            // fail to authenticate
        }
```
如果希望 Spring Security 来处理认证失败，则不需要捕捉 ServletException 。


## HttpServletRequest.logout()
HttpServletRequest.logout() 用于登出。一般会清理 SecurityContextHolder, invalidate HttpSession, 清理 "Remember Me" 认证等等，依赖配置的 LogoutHandler 。在调用  HttpServletRequest.logout() 之后，还是需要输出 response ，一般是转向到 welcome 页面。


## AsyncContext.start(Runnable)
AsyncContext.start(Runnable) 保证在处理 Runnable 时仍然使用当前 SecurityContext ，例如：
```java
        final AsyncContext async = httpServletRequest.startAsync();
        async.start(new Runnable() {
            public void run() {
                Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
                try {
                    final HttpServletResponse asyncResponse = (HttpServletResponse) async.getResponse();
                    asyncResponse.setStatus(HttpServletResponse.SC_OK);
                    asyncResponse.getWriter().write(String.valueOf(authentication));
                    async.complete();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        });
```


## Async Servlet Support
FIXME Async Servlet Support


# Servlet 3.1+ Integration
## HttpServletRequest.changeSessionId()
HttpServletRequest.changeSessionId() 是 Spring Security 3.1+ 中预防 Session Fixation 攻击的默认方法。

