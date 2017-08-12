# Constructor-based dependency injection
## Constructor argument resolution
No potential ambiguity exists 的例子


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="foo" class="org.example.demo.spring.context.Test.Foo">
    <constructor-arg ref="baz" />
    <constructor-arg ref="bar" />
  </bean>
  <bean id="bar" class="org.example.demo.spring.context.Test.Bar" />
  <bean id="baz" class="org.example.demo.spring.context.Test.Baz" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$Foo@1fc2b765
        System.out.println(applicationContext.getBean("foo"));
        applicationContext.close();
    }

    public static class Foo {
        public Foo(Bar bar, Baz baz) {
            // org.example.demo.spring.context.Test$Bar@757942a1
            System.out.println(bar);
            // org.example.demo.spring.context.Test$Baz@4a87761d
            System.out.println(baz);
        }
    }

    public static class Bar {
    }

    public static class Baz {
    }
}
```


### constructor-arg type
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="exampleBean" class="org.example.demo.spring.context.Test.ExampleBean">
    <constructor-arg type="java.lang.String" value="42" />
    <constructor-arg type="int" value="7500000" />
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        ExampleBean bean = applicationContext.getBean(ExampleBean.class);
        // 7500000
        System.out.println(bean.years);
        // 42
        System.out.println(bean.ultimateAnswer);
        applicationContext.close();
    }

    public static class ExampleBean {
        private int years;
        private String ultimateAnswer;

        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
}
```


如果去掉 test.xml 第 6,7 行的 type 则结果刚好相反。如果第 6,7 行只有一行去除 type ，则结果还是对的。


### constructor-arg index
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="exampleBean" class="org.example.demo.spring.context.Test.ExampleBean">
    <constructor-arg index="0" value="7500000" />
    <constructor-arg index="1" value="42" />
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        ExampleBean bean = applicationContext.getBean(ExampleBean.class);
        // 7500000
        System.out.println(bean.years);
        // 42
        System.out.println(bean.ultimateAnswer);
        applicationContext.close();
    }

    public static class ExampleBean {
        private int years;
        private String ultimateAnswer;

        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
}
```


同样， test.xml 第 6,7 行只需要有一行指定 index 即可。


### constructor-arg name
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="exampleBean" class="org.example.demo.spring.context.Test.ExampleBean">
    <constructor-arg name="years" value="7500000" />
    <constructor-arg name="ultimateAnswer" value="42" />
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        ExampleBean bean = applicationContext.getBean(ExampleBean.class);
        // 7500000
        System.out.println(bean.years);
        // 42
        System.out.println(bean.ultimateAnswer);
        applicationContext.close();
    }

    public static class ExampleBean {
        private int years;
        private String ultimateAnswer;

        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
}
```


### @ConstructorProperties
Keep in mind that to make this work out of the box your code must be compiled with the debug flag enabled so that Spring can look up the parameter name from the constructor. If you can’t compile your code with debug flag (or don’t want to) you can use @ConstructorProperties JDK annotation to explicitly name your constructor arguments.


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="exampleBean" class="org.example.demo.spring.context.Test.ExampleBean">
    <constructor-arg name="ua" value="42" />
    <constructor-arg name="y" value="7500000" />
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import java.beans.ConstructorProperties;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        ExampleBean bean = applicationContext.getBean(ExampleBean.class);
        // 7500000
        System.out.println(bean.years);
        // 42
        System.out.println(bean.ultimateAnswer);
        applicationContext.close();
    }

    public static class ExampleBean {
        private int years;
        private String ultimateAnswer;

        @ConstructorProperties({ "y", "ua" })
        public ExampleBean(int years, String ultimateAnswer) {
            this.years = years;
            this.ultimateAnswer = ultimateAnswer;
        }
    }
}
```


相关源码参见 org.springframework.beans.factory.support.ConstructorResolver 。


# Setter-based dependency injection
# Dependency resolution process
circular dependency


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <constructor-arg name="b" ref="b" />
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B">
    <constructor-arg name="a" ref="a" />
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        // org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        A bean = applicationContext.getBean(A.class);
        System.out.println(bean);
        applicationContext.close();
    }

    public static class A {
        public A(B b) {
        }
    }

    public static class B {
        public B(A a) {
        }
    }
}
```
