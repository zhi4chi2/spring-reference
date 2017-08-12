# 术语
- configuration metadata - represented in XML, Java annotations, or Java code. It allows you to express the objects that compose your application and the rich interdependencies between such objects.
- bean - an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.
- factory bean - a bean that is configured in the Spring container that will create objects through an instance or static factory method.
- FactoryBean - a Spring-specific FactoryBean


# modules
BOM(Maven Repo 中也一致)
- spring-aop - 基于代理的 AOP 支持
- spring-aspects - 基于 AspectJ 的 aspects
- spring-beans - Beans 支持，包括 Groovy
- spring-context - 包括 scheduling and remoting 抽象
- spring-context-support
- spring-core
- spring-expression
- spring-instrument - Instrumentation agent for JVM bootstrapping
- spring-instrument-tomcat - Instrumentation agent for Tomcat
- spring-jdbc - 包括 DataSource setup and JDBC access 支持
- spring-jms - 包括 helper classes to send and receive JMS messages
- spring-messaging - messaging architectures and protocols 支持
- spring-orm
- spring-oxm
- spring-test
- spring-tx - 包括 DAO support and JCA 集成
- spring-web - 包括 client and web remoting
- spring-webmvc - 包括 REST Web Services
- spring-webmvc-portlet
- spring-websocket - WebSocket and SockJS 实现，包括 STOMP 支持


# Reference
- http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/ - html 版本
- http://repo.spring.io/release/org/springframework/spring/4.3.9.RELEASE/spring-framework-4.3.9.RELEASE-docs.zip 中有 pdf/epub 版本
- https://spring.io/guides
