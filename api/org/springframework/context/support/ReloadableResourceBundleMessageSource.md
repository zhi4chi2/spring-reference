# fields
## fallbackToSystemLocale
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basename" value="classpath:i18n/messages" />
  </bean>
</beans>
```


src/main/resources/i18n/messages.properties
```
test=english test
```


src/main/resources/i18n/messages\_zh\_CN.properties
```
test=\u4E2D\u6587\u6D4B\u8BD5
```


```java
package org.example.demo.spring.context;

import java.util.Locale;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/test.xml");
        // 中文测试
        System.out.println(context.getMessage("test", null, Locale.ENGLISH));
        context.close();
    }
}
```


如果修改 test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basename" value="classpath:i18n/messages" />
    <property name="fallbackToSystemLocale" value="false" />
  </bean>
</beans>
```


再次执行 Test 则输出 `english test`
