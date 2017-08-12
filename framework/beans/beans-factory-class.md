# Instantiation with a static factory method
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" factory-method="createInstance" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.ArrayList;
import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // class java.util.ArrayList
        System.out.println(applicationContext.getBean("a").getClass());
        applicationContext.close();
    }

    public static class A {
        static List<String> createInstance() {
            List<String> list = new ArrayList<>();
            return list;
        }
    }
}
```


# Instantiation using an instance factory method
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" />
  <bean id="b" factory-bean="a" factory-method="createInstance" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.ArrayList;
import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // class java.util.ArrayList
        System.out.println(applicationContext.getBean("b").getClass());
        applicationContext.close();
    }

    public static class A {
        List<String> createInstance() {
            List<String> list = new ArrayList<>();
            return list;
        }
    }
}
```
