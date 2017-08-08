# The AuthenticationManager, ProviderManager and AuthenticationProvider
- AuthenticationManager - 接口
- ProviderManager - AuthenticationManager 的默认实现，具体认证工作委托给一系列 AuthenticationProvider ，如果所有 AuthenticationProvider 都【返回 null 或者抛出 AuthenticationException 】，则抛出 ProviderNotFoundException 异常（都返回 null 时）或者 AuthenticationException 异常（有 AuthenticationProvider 抛出 AuthenticationException 时）。
- AuthenticationProvider - 做具体的认证工作，或者返回一个 fully populated Authentication object ，或者抛出异常(authentication fails)，或者返回 null(AuthenticationProvider is unable to support authentication of the passed Authentication object)。
- DaoAuthenticationProvider - AuthenticationProvider 的一个实现，使用 UserDetailsService 。


使用命名空间时， ProviderManager 在内部自动创建。如果没有使用命名空间，则创建 AuthenticationManager 的 bean 定义示例如下：
```xml
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:security="http://www.springframework.org/schema/security"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <bean id="authenticationManager" class="org.springframework.security.authentication.ProviderManager">
    <constructor-arg>
      <list>
        <ref local="daoAuthenticationProvider" />
        <ref local="anonymousAuthenticationProvider" />
        <ref local="ldapAuthenticationProvider" />
      </list>
    </constructor-arg>
  </bean>
  <bean id="daoAuthenticationProvider" class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
    <property name="userDetailsService" ref="inMemoryDaoImpl" />
    <property name="passwordEncoder" ref="passwordEncoder" />
  </bean>
</beans>
```


不同的 authentication mechanisms 需要的 AuthenticationProvider 可能是不一样的，比如 form-login 和 CAS 需要的 AuthenticationProvider 就不一样。每个 AuthenticationProvider 都有自己支持的 Authentication 类，参见 `AuthenticationProvider.supports(Class&lt;?> authentication)` 。比如 CasAuthenticationToken 就需要 CasAuthenticationProvider 。


## Erasing Credentials on Successful Authentication
从 3.1+ 开始， ProviderManager 会试图在返回的 Authentication 对象上清除 credentials 等敏感信息。如果不希望如此，可以配置 ProviderManager.eraseCredentialsAfterAuthentication 属性为 false 。


## DaoAuthenticationProvider


# UserDetailsService Implementations
大多数 AuthenticationProvider 都会使用 UserDetailsService ，即使不是为了认证（而只是为了取 GrantedAuthority 信息，认证由其它系统，例如 LDAP, X.509, CAS 等完成）。


UserDetailsService 的内建实现：
- InMemoryUserDetailsManager
- JdbcDaoImpl/JdbcUserDetailsManager


## In-Memory Authentication
内存认证，常用于 prototype 应用：
```xml
  <user-service id="userDetailsService">
    <user name="jimi" password="jimispassword" authorities="ROLE_USER, ROLE_ADMIN" />
    <user name="bob" password="bobspassword" authorities="ROLE_USER" />
  </user-service>
```


或
```xml
<user-service id="userDetailsService" properties="users.properties"/>
```


属性文件格式：
```
username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]
```


例如：
```
jimi=jimispassword,ROLE_USER,ROLE_ADMIN,enabled
```


## JdbcDaoImpl
JdbcDaoImpl 的例子
```xml
  <bean id="userDetailsService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <property name="dataSource" ref="dataSource" />
  </bean>
```


By default, JdbcDaoImpl loads the authorities for a single user with the assumption that the authorities are mapped directly to users (see the database schema appendix). An alternative approach is to partition the authorities into groups and assign groups to the user. Some people prefer this approach as a means of administering user rights. See the JdbcDaoImpl Javadoc for more information on how to enable the use of group authorities. The group schema is also included in the appendix.


# Password Encoding
org.springframework.security.crypto.password.PasswordEncoder 是新的接口， org.springframework.security.authentication.encoding.PasswordEncoder 是旧的接口。 DaoAuthenticationProvider 中既可以使用新的接口，也可以使用旧的接口。参见 `DaoAuthenticationProvider.setPasswordEncoder(Object passwordEncoder)` 。


推荐使用 org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder 而不是 MD5 or SHA


文档中说
- one-way password hashing algorithm such as bcrypt
- plain hash function such as MD5 or SHA


bcrypt 使用 built-in salt value ，对每个密码都不一样。 Bcrypt 故意被设计得很慢，以避免暴力破解。而 MD5/SHA 很快。


如果 application 已经使用了 MD5/SHA ，想更换为 bcrypt ，则需要通知所有用户更换密码，并在更换期间自定义 PasswordEncoder 使两种皆可。很麻烦。


## What is a hash
- hash algorithm - A hash (or digest) algorithm is a one-way function which produces a piece of fixed-length output data (the hash) from some input data


## Adding Salt to a Hash
弱密码容易被暴力破解，解决方法是使用 salt 。先将 password 与 salt 组合在一起，然后再计算 hash 。


Bcrypt 自动对每个 password 生成随机的 salt ，并用标准格式保存在 bcrypt string 中。因此不需要手动处理 salt 。


传统的方式是在 DaoAuthenticationProvider 中注入 SaltSource ， SaltSource 从 UserDetails 中得到 salt 值。 bcrypt 不需要如此，但如果使用遗留的系统，仍然可以这样配置 SaltSource


## Hashing and Authentication
当使用 PasswordEncoder 时，数据库中保存的密码也需要被编码，并且大小写也要一致。可以使用 PasswordEncoder.encode() 得到编码后的值。


# Jackson Support
Spring Security has added Jackson Support for persisting Spring Security related classes. This can improve the performance of serializing Spring Security related classes when working with distributed sessions (i.e. session replication, Spring Session, etc).


例如
```java
ObjectMapper mapper = new ObjectMapper();
ClassLoader loader = getClass().getClassLoader();
List<Module> modules = SecurityJackson2Modules.getModules(loader);
mapper.registerModules(modules);

// ... use ObjectMapper as normally ...
SecurityContext context = new SecurityContextImpl();
// ...
String json = mapper.writeValueAsString(context);
```
