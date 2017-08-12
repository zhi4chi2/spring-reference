src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
  default-destroy-method="destroy">
  <bean id="a" class="org.example.demo.spring.context.Test.A" depends-on="b,c" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.C" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.registerShutdownHook();
        applicationContext.close();
    }

    public static class A {
        public A() {
            System.out.println("A");
        }

        public void destroy() {
            System.out.println("destroy a");
        }
    }

    public static class B {
        public B() {
            System.out.println("B");
        }

        public void destroy() {
            System.out.println("destroy b");
        }
    }

    public static class C {
        public C() {
            System.out.println("C");
        }

        public void destroy() {
            System.out.println("destroy c");
        }
    }
}
```


输出
```
B
C
A
destroy a
destroy c
destroy b
```
