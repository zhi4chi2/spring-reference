- [Spring Security](/security/README.md)
  - [Preface](/security/preface/README.md)
    - Getting Started
    - [Introduction](/security/preface/introduction.md)
    - [What's New in Spring Security 4.2](/security/preface/whats-new.md)
    - [Samples and Guides](/security/preface/samples.md)
    - [Java Configuration](/security/preface/java-config.md)
    - [Security Namespace Configuration](/security/preface/namespace/README.md)
      - [A Minimal http Configuration](/security/preface/namespace/minimal-http.md)
      - [form login](/security/preface/namespace/form-login.md)


At the bottom level you'll need to deal with issues such as transport security and system identification, in order to mitigate man-in-the-middle attacks.


Next you'll generally utilise firewalls, perhaps with VPNs or IP security to ensure only authorised systems can attempt to connect.


In corporate environments you may deploy a DMZ to separate public-facing servers from backend database and application servers.


Your operating system will also play a critical part, addressing issues such as running processes as non-privileged users and maximising file system security. An operating system will usually also be configured with its own firewall. Hopefully somewhere along the way you'll be trying to prevent denial of service and brute force attacks against the system.


An intrusion detection system will also be especially useful for monitoring and responding to attacks, with such systems able to take protective action such as blocking offending TCP/IP addresses in real-time.


Moving to the higher layers, your Java Virtual Machine will hopefully be configured to minimize the permissions granted to different Java types, and then your application will add its own problem domain-specific security configuration. Spring Security makes this latter area - application security - much easier.


you will need to properly address all security layers mentioned above, together with managerial factors that encompass every layer. A non-exhaustive list of such managerial factors would include security bulletin monitoring, patching, personnel vetting, audits, change control, engineering management systems, data backup, disaster recovery, performance benchmarking, load monitoring, centralised logging, incident response procedures etc.


if you are building a web application, you should be aware of the many potential vulnerabilities such as cross-site scripting, request-forgery and session-hijacking which you should be taking into account from the start. The OWASP web site (http://www.owasp.org/) maintains a top ten list of web application vulnerabilities as well as a lot of useful reference information.


http://www.owasp.org/ 列出 web application 易受攻击的地方，比如：
*cross-site scripting
*request-forgery
*session-hijacking
