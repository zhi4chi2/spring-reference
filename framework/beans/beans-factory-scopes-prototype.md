src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" scope="prototype" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.registerShutdownHook();
        applicationContext.getBean(A.class);
        applicationContext.close();
    }

    public static class A implements InitializingBean, DisposableBean {
        @Override
        public void destroy() throws Exception {
            System.out.println("A.destroy");
        }

        @Override
        public void afterPropertiesSet() throws Exception {
            System.out.println("A.afterPropertiesSet");
        }
    }
}
```


运行输出
```
A.afterPropertiesSet
```


如果将 test.xml 中的 `scope="prototype"` 去掉，则输出
```
A.afterPropertiesSet
A.destroy
```
