src/main/webapp/index.jsp
```jsp
<!DOCTYPE html>
<html>
<head>
<title>Index</title>
</head>
<body>
Hello, world!
</body>
</html>
```


# authentication-provider.user-service-ref
src/main/resources/spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
  </http>

  <authentication-manager>
    <authentication-provider user-service-ref="myUserDetailsService" />
  </authentication-manager>

  <beans:bean id="myUserDetailsService" class="org.example.demo.spring.security.MyUserDetailsService" />
</beans:beans>
```


```java
package org.example.demo.spring.security;

import java.util.Collections;

import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return new User(username, "pw", Collections.singleton(() -> "ROLE_USER"));
    }
}
```


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ ，登入，进入 /


# jdbc-user-service
更改 pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.example.demo</groupId>
    <artifactId>demo-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>

  <artifactId>demo-spring-security</artifactId>
  <packaging>war</packaging>

  <name>demo-spring-security</name>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-bom</artifactId>
        <version>4.2.3.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.taglibs</groupId>
      <artifactId>taglibs-standard-spec</artifactId>
      <version>1.2.5</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.taglibs</groupId>
      <artifactId>taglibs-standard-impl</artifactId>
      <version>1.2.5</version>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-acl</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
    </dependency>

    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <version>9.4.1212</version>
    </dependency>
    <dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>1.4</version>
    </dependency>

    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```


更改 spring-security.xml
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
  </http>

  <authentication-manager>
    <authentication-provider>
      <jdbc-user-service data-source-ref="dataSource" />
    </authentication-provider>
  </authentication-manager>

  <beans:bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <beans:property name="driverClassName" value="org.postgresql.Driver" />
    <beans:property name="url" value="jdbc:postgresql://127.0.0.1:5432/test" />
    <beans:property name="username" value="postgres" />
    <beans:property name="password" value="postgres" />
  </beans:bean>
</beans:beans>
```


新建数据库 test 并执行下面 SQL
```sql
create table users(
	username varchar(50) not null primary key,
	password varchar(50) not null,
	enabled boolean not null
);

create table authorities (
	username varchar(50) not null,
	authority varchar(50) not null,
	constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index ix_auth_username on authorities (username,authority);
insert into users(username, password, enabled) values('jimi', 'pw', true);
insert into authorities(username, authority) values('jimi', 'ROLE_USER');
```


测试：
- 浏览器访问 http://localhost:8080/demo-spring-security/ 登入


将 spring-security.xml 改为
```xml
<beans:beans xmlns="http://www.springframework.org/schema/security" xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">
  <http>
    <intercept-url pattern="/**" access="hasRole('USER')" />
    <form-login />
  </http>

  <authentication-manager>
    <authentication-provider user-service-ref='myUserDetailsService' />
  </authentication-manager>

  <beans:bean id="myUserDetailsService" class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
    <beans:property name="dataSource" ref="dataSource" />
  </beans:bean>

  <beans:bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <beans:property name="driverClassName" value="org.postgresql.Driver" />
    <beans:property name="url" value="jdbc:postgresql://127.0.0.1:5432/test" />
    <beans:property name="username" value="postgres" />
    <beans:property name="password" value="postgres" />
  </beans:bean>
</beans:beans>
```


效果一样。


