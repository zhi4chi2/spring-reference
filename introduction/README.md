# Dependency Injection and Inversion of Control
# Modules
- Core Container
  - Core
  - Beans
  - Context
  - SpEL
- AOP
- Aspects
- Instrumentation
- Messaging
- Data Access/Integration
  - JDBC
  - ORM
  - OXM
  - JMS
  - Transactions
- Web
  - Web
  - Servlet
  - Portlet
  - WebSocket
- Test


## Core Container
*spring-core
*spring-beans
*spring-context
*spring-context-support
*spring-expression


spring-core 和 spring-beans 是核心。支持依赖注入特性。


spring-context 可以通过 framework-style 的方式存取对象，就像 JNDI 一样。在 spring-beans 的基础上添加了
*国际化
*事件传播
*资源装载
*透明创建上下文（例如 Servlet container ）
*支持 EJB, JMX, and basic remoting


spring-context-support 集成第三方 jar ，包括：
*caching (EhCache, Guava, JCache)
*mailing (JavaMail)
*scheduling (CommonJ, Quartz)
*template engines (FreeMarker, JasperReports, Velocity)


spring-expression 提供在运行时操纵对象图的 EL 语言。是 JSP 2.1 unified EL 的扩展。


## AOP and Instrumentation
spring-aop 提供了遵从 AOP 联盟的 AOP 实现。


spring-aspects 用于与 AspectJ 集成。


spring-instrument 提供 class instrumentation 支持以及 classloader 实现，用于特定的 application server 。


spring-instrument-tomcat 包含用于 tomcat 的 Spring instrumentation agent 。


## Messaging
spring-messaging 包含 Spring Integration 项目中的核心概念，包括 Message, MessageChannel, MessageHandler 等等，用于基于消息(messaging-based)的应用程序。还有一些注解，用于映射 messages 到方法。


## Data Access/Integration
*spring-jdbc
*spring-tx
*spring-orm - 与 ORM 工具集成，包括 JPA, JDO, and Hibernate
*spring-oxm - 支持 Object/XML mapping 的实现： JAXB, Castor, XMLBeans, JiBX and XStream
*spring-jms - 创造和消费 messages 。从 4.1 开始，提供与 spring-messaging 的集成


## Web
*spring-web
*spring-webmvc
*spring-websocket
*spring-webmvc-portlet


spring-web 提供面向 web 的集成特性，例如文件上传，使用 Servlet listeners 初始化 IoC container ，以及面向 web 的 application context 。还包含 HTTP client ，和 web 相关的 remoting 支持。


spring-webmvc 包含 MVC 和 REST Web Services 实现。


spring-webmvc-portlet 包含 Portlet 环境下的 MVC 实现。


## Test
支持 JUnit or TestNG 。还包括 mock objects ，用于测试。


# Usage scenarios
## Dependency Management and Naming Conventions
*spring-aop - 基于代理的 AOP 支持
*spring-aspects - 基于 AspectJ 的 aspects
*spring-beans - Beans 支持，包括 Groovy
*spring-context - 包括 scheduling and remoting 抽象
*spring-context-support
*spring-core
*spring-expression
*spring-instrument - Instrumentation agent for JVM bootstrapping
*spring-instrument-tomcat - Instrumentation agent for Tomcat
*spring-jdbc - 包括 DataSource setup and JDBC access 支持
*spring-jms - 包括 helper classes to send and receive JMS messages
*spring-messaging - messaging architectures and protocols 支持
*spring-orm
*spring-oxm
*spring-test
*spring-tx - 包括 DAO support and JCA 集成
*spring-web - 包括 client and web remoting
*spring-webmvc - 包括 REST Web Services
*spring-webmvc-portlet
*spring-websocket - WebSocket and SockJS 实现，包括 STOMP 支持


如果项目有个第三方 jar 依赖某个 spring artifact ，而该 spring artifact 并没有在项目的 pom.xml 中显式声明，则此传递依赖的 spring artifact 的版本可能与项目中其它 spring artifact 的版本不一致，因此可能导致错误。解决办法是在项目的 pom.xml 中预声明所有 spring artifact 的版本，但这比较繁琐，还有种办法就是 import spring-framework-bom
```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-framework-bom</artifactId>
        <version>4.3.9.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```


例如若有
```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-framework-bom</artifactId>
        <version>4.1.7.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>4.2.1.RELEASE</version>
    </dependency>
  </dependencies>
```
则 spring-context-support 的版本是 4.2.1.RELEASE ，但引入的传递依赖 spring-beans 等等本应是 4.2.1.RELEASE 但最后都是 4.1.7.RELEASE 版本（ dependencyManagement 最终起作用）。一般情况下，第 17 行的版本可以省略以保持所有 artifact 版本一致。


FIXME Gradle 与 Ivy 的配置


完整发布版可以从 http://repo.spring.io/release/org/springframework/spring/ 下载。
- http://repo.spring.io/release/org/springframework/spring/4.3.9.RELEASE/spring-framework-4.3.9.RELEASE-dist.zip
- http://repo.spring.io/release/org/springframework/spring/4.3.9.RELEASE/spring-framework-4.3.9.RELEASE-docs.zip - html/pdf/epub
- http://repo.spring.io/release/org/springframework/spring/4.3.9.RELEASE/spring-framework-4.3.9.RELEASE-schema.zip - xsd


## Logging
Spring 使用 Jakarta Commons Logging API 记录日志。 Spring 的代码中使用了 JCL 的类，并且其中的 Log 对象暴露给子类使用。因此如果自定义扩展 Spring ，可能也用到了 Spring 代码中的 Log 对象。所以，期望对于扩展了 Spring 的项目，在升级 Spring 版本时依然可用，因此 Spring 新版本依然依赖 JCL 。


在应用服务器中运行 Spring 程序时，应用服务器可能有自己的 JCL 实现，比如 IBM Websphere Application Server (WAS) 。解决方法见原文档，好像很麻烦。
