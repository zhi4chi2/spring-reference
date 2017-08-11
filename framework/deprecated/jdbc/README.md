- Data Access
  - [Data access with JDBC](/framework/jdbc/README.md)
    - [Introduction to Spring Framework JDBC](/framework/jdbc/01.md)
    - [Using the JDBC core classes to control basic JDBC processing and error handling](/framework/jdbc/02/README.md)
      - [JdbcTemplate](/framework/jdbc/02/01.md)
      - [NamedParameterJdbcTemplate](/framework/jdbc/02/02.md)


# 测试项目
pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.example.demo</groupId>
    <artifactId>demo-spring</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>

  <artifactId>demo-spring-tx</artifactId>
  <packaging>jar</packaging>

  <name>demo-spring-tx</name>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
    </dependency>

    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <version>9.4.1212</version>
    </dependency>
    <dependency>
      <groupId>p6spy</groupId>
      <artifactId>p6spy</artifactId>
      <version>2.1.4</version>
    </dependency>
  </dependencies>
</project>
```


src/main/resources/spy.properties
```
databaseDialectDateFormat=yyyy-MM-dd HH:mm:ss
dateformat=yyyy-MM-dd HH:mm:ss
append=false
logMessageFormat=org.example.demo.SimpleMessageFormat
```


```java
package org.example.demo;

import com.p6spy.engine.common.P6Util;
import com.p6spy.engine.spy.appender.MessageFormattingStrategy;

public class SimpleMessageFormat implements MessageFormattingStrategy {
    @Override
    public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared, String sql) {
        if (sql == null || sql.trim().isEmpty()) {
            return P6Util.singleLine(prepared);
        } else {
            return P6Util.singleLine(sql);
        }
    }
}
```


src/main/resources/logback.xml
```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>[%thread] %-5level %logger- %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="org.springframework.jdbc" level="debug" />

  <root level="info">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

