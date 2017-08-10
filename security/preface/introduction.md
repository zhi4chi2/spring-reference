# What is Spring Security
Java EE's Servlet Specification or EJB Specification are not portable at a WAR or EAR level


authorization capabilities
- authorizing web requests - consider the authorization capabilities found in the Servlet Specification web pattern security
- authorizing whether methods can be invoked - consider EJB Container Managed Security
- authorizing access to individual domain object instances - consider file system security


# History
Acegi Security became an official Spring Portfolio project towards the end of 2007 and was rebranded as "Spring Security".


# Release Numbering
MAJOR.MINOR.PATCH


MAJOR versions are incompatible, large-scale upgrades of the API.


MINOR versions should largely retain source and binary compatibility with older minor versions, thought there may be some design changes and incompatible updates.


PATCH level should be perfectly compatible, forwards and backwards, with the possible exception of changes which are to fix bugs and defects.


# Getting Spring Security
## Usage with Maven
Spring Security builds against Spring Framework 4.3.9.RELEASE, but should work with 4.0.x.


## Gradle
FIXME


## Project Modules
- spring-security-core.jar - core authentication and access-contol classes and interfaces, remoting support and basic provisioning APIs. Supports standalone applications, remote clients, method (service layer) security and JDBC user provisioning(使用 JDBC 取得用户信息).
- spring-security-remoting.jar - 与 Spring Remoting 集成。当使用 Spring Remoting 编写 remote client 时使用
- spring-security-web.jar - filters and related web-security infrastructure code, 基于 URL 的访问控制，基于 web 的认证
- spring-security-config.jar - 解析 security 命名空间，其中的类不应该在 application 中直接使用
- spring-security-ldap.jar - LDAP 认证和管理 LDAP 条目
- spring-security-acl.jar - domain object ACL
- spring-security-cas.jar - 使用 CAS 认证
- spring-security-openid.jar - OpenID 认证，通过外部 OpenID server 认证用户。需要 OpenID4Java
- spring-security-test.jar - Support for testing with Spring Security, 4.0 新增


Maven repo 中的其它 artifact
- spring-security-crypto
- spring-security-taglibs
- spring-security-aspects
- spring-security-data - 4.0 新增
- spring-security-messaging - 4.0 新增
- spring-security-samples-javaconfig-messages - 不在 BOM 中， 4.1.0.RELEASE 新增
- spring-security-samples-messages-jc - 不在 BOM 中，最新版 4.0.4.RELEASE


## Checking out the Source
得到源代码(包括 samples)
```
git clone https://github.com/spring-projects/spring-security.git
```


samples 在 https://github.com/spring-projects/spring-security/tree/4.2.3.RELEASE/samples

