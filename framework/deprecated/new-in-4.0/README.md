Spring 2.0 提供 XML 命名空间和 AspectJ 支持。


Spring 2.5 提供注解驱动的配置。


Spring 3.0 基于 Java 5+ 并支持 Java-based @Configuration 配置。


Spring 4.0 完整支持 Java 8 。最低需要 Java SE 6


# Improved Getting Started Experience
https://spring.io/guides 提供了很多向导用于学习 Spring 。


# Removed Deprecated Packages and Methods
废弃的 package, class, method 被删除。完整的列表见 http://docs.spring.io/spring-framework/docs/3.2.4.RELEASE_to_4.0.0.RELEASE/


Spring 4.0 依赖的第三方 jar 需要是 2010 年之后发布的版本， 2010 年之前的版本不再支持。例如 Hibernate 3.6+, EhCache 2.1+, Quartz 1.8+, Groovy 1.8+, and Joda-Time 2.0+ 。但有些例外：
*Hibernate Validator 4.3+ - 必须是 4.3+ ，即使在 2010 年之后发布的其它版本也不行
*Jackson 2.0+ - 1.8/1.9 在 Spring 3.2 中支持


# Java 8
支持 Java 8 的新特性，例如：
*lambda expressions - 用于 Spring callback 接口
*method references - 用于 Spring callback 接口
*支持 java.time(JSR-310)
*部分注解被标为 @Repeatable
*基于 -parameters 编译选项，使用 parameter name discovery


最小支持 JDK 6 update 18(2010 年 1 月发布)


# Java EE 6 and 7
Spring 4 需要 Java EE 6+ ，支持 JPA 2.0 and Servlet 3.0 。虽然还能发布到 Servlet 2.5 环境，但 Spring 4 是基于 Servlet 3.0+ 做的测试。


如果使用 WebSphere 7 则需要安装 JPA 2.0 feature pack 。如果使用 WebLogic 10.3.4+ ，需要安装 JPA 2.0 patch 。


Spring 4.0 支持 JavaEE 7 的特性： JMS 2.0, JTA 1.2, JPA 2.1, Bean Validation 1.1, and JSR-236 Concurrency Utilities


Hibernate 4.3 因为是 JPA 2.1 provider ，所以从 Spring 4.0 才开始支持。同理， Hibernate Validator 5.0 是 Bean Validation 1.1 provider 所以也是从 Spring 4.0 才开始支持。而 Spring 3.2 版不支持它们。


# Groovy Bean Definition DSL
从 4.0 开始支持用 Groovy DSL 定义 bean 。因此支持在 bootstrap code 中嵌入 bean 定义。


参见 http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/groovy/GroovyBeanDefinitionReader.html


# Core Container Improvements
*在注入 bean 时，将泛型视为一种 qualifier ，参见 Using generics as autowiring qualifiers
*参见 Meta-annotations


FIXME

