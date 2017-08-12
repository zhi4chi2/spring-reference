> The id attribute allows you to specify exactly one id. Conventionally these names are alphanumeric ('myBean', 'fooService', etc.), but may contain special characters as well. If you want to introduce other aliases to the bean, you can also specify them in the name attribute, separated by a comma (,), semicolon (;), or white space.


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" name="b,c;d e" class="org.example.demo.spring.context.Test.A" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$A@371a67ec
        System.out.println(applicationContext.getBean("a"));
        // org.example.demo.spring.context.Test$A@371a67ec
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$A@371a67ec
        System.out.println(applicationContext.getBean("c"));
        // org.example.demo.spring.context.Test$A@371a67ec
        System.out.println(applicationContext.getBean("d"));
        // org.example.demo.spring.context.Test$A@371a67ec
        System.out.println(applicationContext.getBean("e"));
        applicationContext.close();
    }

    public static class A {
    }
}
```


> As a historical note, in versions prior to Spring 3.1, the id attribute was defined as an xsd:ID type, which constrained possible characters. As of 3.1, it is defined as an xsd:string type. Note that bean id uniqueness is still enforced by the container, though no longer by XML parsers.


参见 spring-beans-3.0.xsd 第 50 行(`type="xsd:ID"`)和 spring-beans-3.1.xsd 第 50 行(`type="xsd:string"`)。


> You are not required to supply a name or id for a bean. If no name or id is supplied explicitly, the container generates a unique name for that bean.


相关源码见 org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionElement 方法。其中调用的 checkNameUniqueness 保证 bean name 和 所有 alias 在容器中都是唯一的。


> Spring generates bean names for unnamed components, following the rules above: essentially, taking the simple class name and turning its initial character to lower-case.


但实测不是这样的！


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean class="org.example.demo.spring.context.Test.A" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Arrays;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // [org.example.demo.spring.context.Test.A#0]
        System.out.println(Arrays.toString(applicationContext.getBeanDefinitionNames()));
        applicationContext.close();
    }

    public static class A {
    }
}
```


相关源码见 org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionElement 方法。


# Aliasing a bean outside the bean definition
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <alias name="a" alias="b" />
  <bean id="a" class="org.example.demo.spring.context.Test.A" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$A@50d0686
        System.out.println(applicationContext.getBean("a"));
        // org.example.demo.spring.context.Test$A@50d0686
        System.out.println(applicationContext.getBean("b"));
        applicationContext.close();
    }

    public static class A {
    }
}
```

