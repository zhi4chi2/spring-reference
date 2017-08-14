# Lookup method injection
Spring Framework implements this method injection by using bytecode generation from the CGLIB library to generate dynamically a subclass that overrides the method.


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="commandManager" class="org.example.demo.spring.context.Test.CommandManager">
    <lookup-method name="createCommand" bean="myCommand" />
  </bean>
  <bean id="myCommand" class="org.example.demo.spring.context.Test.Command" scope="prototype" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        CommandManager cm = applicationContext.getBean(CommandManager.class);
        // class org.example.demo.spring.context.Test$CommandManager$$EnhancerBySpringCGLIB$$3f6647fa
        System.out.println(cm.getClass());

        // org.example.demo.spring.context.Test$Command@25359ed8
        cm.process(null);
        // org.example.demo.spring.context.Test$Command@21a947fe
        cm.process(null);
        applicationContext.close();
    }

    public static abstract class CommandManager {
        public Object process(Object commandState) {
            Command command = createCommand();
            System.out.println(command);
            command.setState(commandState);
            return command.execute();
        }

        protected abstract Command createCommand();
    }

    public final static class Command {
        public Object execute() {
            return null;
        }

        public void setState(Object obj) {
        }
    }
}
```
___


更改 createCommand 方法
```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        CommandManager cm = applicationContext.getBean(CommandManager.class);
        // class org.example.demo.spring.context.Test$CommandManager$$EnhancerBySpringCGLIB$$3f6647fa
        System.out.println(cm.getClass());

        // org.example.demo.spring.context.Test$Command@25359ed8
        cm.process(null);
        // org.example.demo.spring.context.Test$Command@21a947fe
        cm.process(null);
        applicationContext.close();
    }

    public static class CommandManager {
        public Object process(Object commandState) {
            Command command = createCommand();
            System.out.println(command);
            command.setState(commandState);
            return command.execute();
        }

        protected Command createCommand() {
            return null;
        }
    }

    public final static class Command {
        public Object execute() {
            return null;
        }

        public void setState(Object obj) {
        }
    }
}
```
也可以
___


将 CommandManager 标为 final 将抛异常 `java.lang.IllegalArgumentException: Cannot subclass final class org.example.demo.spring.context.Test$CommandManager`


将 createCommand 方法标为 final
```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        CommandManager cm = applicationContext.getBean(CommandManager.class);
        // class org.example.demo.spring.context.Test$CommandManager$$EnhancerBySpringCGLIB$$3f6647fa
        System.out.println(cm.getClass());

        // null
        cm.process(null);
        // null
        cm.process(null);
        applicationContext.close();
    }

    public static class CommandManager {
        public Object process(Object commandState) {
            Command command = createCommand();
            System.out.println(command);
            return command;
        }

        public final Command createCommand() {
            return null;
        }
    }

    public final static class Command {
        public Object execute() {
            return null;
        }

        public void setState(Object obj) {
        }
    }
}
```
___


修改 test.xml 去掉 `scope="prototype"`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="commandManager" class="org.example.demo.spring.context.Test.CommandManager">
    <lookup-method name="createCommand" bean="myCommand" />
  </bean>
  <bean id="myCommand" class="org.example.demo.spring.context.Test.Command" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        CommandManager cm = applicationContext.getBean(CommandManager.class);
        // class org.example.demo.spring.context.Test$CommandManager$$EnhancerBySpringCGLIB$$3f6647fa
        System.out.println(cm.getClass());

        // org.example.demo.spring.context.Test$Command@25359ed8
        cm.process(null);
        // org.example.demo.spring.context.Test$Command@25359ed8
        cm.process(null);
        applicationContext.close();
    }

    public static abstract class CommandManager {
        public Object process(Object commandState) {
            Command command = createCommand();
            System.out.println(command);
            command.setState(commandState);
            return command.execute();
        }

        protected abstract Command createCommand();
    }

    public final static class Command {
        public Object execute() {
            return null;
        }

        public void setState(Object obj) {
        }
    }
}
```


## @Lookup
FIXME


# Arbitrary method replacement
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="myValueCalculator" class="org.example.demo.spring.context.Test.MyValueCalculator">
    <replaced-method name="computeValue" replacer="replacementComputeValue">
      <arg-type>String</arg-type>
    </replaced-method>
  </bean>
  <bean id="replacementComputeValue" class="org.example.demo.spring.context.Test.ReplacementComputeValue" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.lang.reflect.Method;

import org.springframework.beans.factory.support.MethodReplacer;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        MyValueCalculator vc = applicationContext.getBean(MyValueCalculator.class);
        // class org.example.demo.spring.context.Test$MyValueCalculator$$EnhancerBySpringCGLIB$$de3678e
        System.out.println(vc.getClass());
        // b
        System.out.println(vc.computeValue("1"));
        // b
        System.out.println(vc.computeValue("2"));
        applicationContext.close();
    }

    public static class MyValueCalculator {
        public String computeValue(String input) {
            return "a";
        }
    }

    public static class ReplacementComputeValue implements MethodReplacer {
        @Override
        public Object reimplement(Object obj, Method method, Object[] args) throws Throwable {
            // 1
            // 2
            System.out.println(args[0]);
            return "b";
        }
    }
}
```


`arg-type` 参见 spring-beans-4.3.9.RELEASE.jar!/org/springframework/beans/factory/xml/spring-beans-4.3.xsd 第 755 行。
