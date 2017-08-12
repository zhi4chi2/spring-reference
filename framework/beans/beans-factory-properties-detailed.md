# Straight values (primitives, Strings, and so on)
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <constructor-arg>
      <value>
        a=aa
        b=bb
      </value>
    </constructor-arg>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Properties;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$A@7ba18f1b
        System.out.println(applicationContext.getBean(A.class));
        applicationContext.close();
    }

    public static class A {
        public A(Properties p) {
            // {b=bb, a=aa}
            System.out.println(p);
        }
    }
}
```


## The idref element
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="theTargetBean" name="targetAlias" class="org.example.demo.spring.context.Test.B" />
  <bean id="theClientBean" class="org.example.demo.spring.context.Test.A">
    <property name="targetName">
      <idref bean="targetAlias" />
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.getBean(A.class);
        applicationContext.close();
    }

    public static class A {
        public void setTargetName(String targetName) {
            // targetAlias
            System.out.println(targetName);
        }
    }

    public static class B {
    }
}
```


如果 test.xml 中第 8 行改为 `<idref bean="notExistBean" />` 则 Test 第 7 行将抛异常 `org.springframework.beans.factory.BeanDefinitionStoreException: Invalid bean name 'notExistBean' in bean reference for bean property 'targetName'`


# References to other beans (collaborators)
## ref bean
reference to any bean in the same container or parent container, regardless of whether it is in the same XML file.


## ref parent
reference to a bean that is in a parent container of the current container.


src/main/resources/spring/parent.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean name="a" class="org.example.demo.spring.context.Test.A" />
</beans>
```


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
      <ref parent="a" />
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext parent = new ClassPathXmlApplicationContext("spring/parent.xml");
        // org.example.demo.spring.context.Test$A@77ec78b9
        System.out.println(parent.getBean(A.class));

        ClassPathXmlApplicationContext child = new ClassPathXmlApplicationContext(new String[] { "spring/test.xml" },
                parent);
        // org.example.demo.spring.context.Test$A@77ec78b9
        System.out.println(child.getBean(A.class));

        // org.springframework.aop.framework.ProxyFactoryBean: 0 interfaces []; 0 advisors []; targetSource [SingletonTargetSource for target object [org.example.demo.spring.context.Test$A@77ec78b9]]; proxyTargetClass=false; optimize=false; opaque=false; exposeProxy=false; frozen=false
        System.out.println(child.getBean("&a"));
        child.close();
        parent.close();
    }

    public static class A {
    }
}
```


这种情况下将 test.xml 第 7 行改为 `<ref bean="a" />` 会报错 `org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': org.springframework.beans.factory.FactoryBeanNotInitializedException: Cannot determine target class for proxy`


如果将 parent.xml 中 bean 改个名字，则 ref bean 也是可以的。


## ref local
> The local attribute on the ref element is no longer supported in the 4.0 beans xsd since it does not provide value over a regular bean reference anymore. Simply change your existing ref local references to ref bean when upgrading to the 4.0 schema.


# Inner beans
> An inner bean definition does not require a defined id or name; if specified, the container does not use such a value as an identifier.


实测不是这样！


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="b">
      <bean id="b" class="org.example.demo.spring.context.Test.B" />
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setB(B b) {
        }
    }

    public static class B implements BeanNameAware {
        @Override
        public void setBeanName(String name) {
            // b
            System.out.println(name);
        }
    }
}
```


跟断点到 `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseBeanDefinitionElement(Element, BeanDefinition)` 也可以看到实际 inner bean 是可以定义 id 的。


# Collections
## list
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="list">
      <list>
        <value>string</value>
        <ref bean="b" />
      </list>
    </property>
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setList(List<Object> l) {
            // [string, org.example.demo.spring.context.Test$B@7276c8cd]
            System.out.println(l);
        }
    }

    public static class B {
    }
}
```


## set
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="set">
      <set>
        <value>string</value>
        <ref bean="b" />
      </set>
    </property>
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Set;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setSet(Set<Object> l) {
            // [string, org.example.demo.spring.context.Test$B@5bfbf16f]
            System.out.println(l);
        }
    }

    public static class B {
    }
}
```


## map
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="map">
      <map>
        <entry>
          <key>
            <value>key1</value>
          </key>
          <value>val1</value>
        </entry>
        <entry>
          <key>
            <value>key2</value>
          </key>
          <ref bean="b" />
        </entry>
      </map>
    </property>
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Map;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setMap(Map<String, Object> l) {
            // {key1=val1, key2=org.example.demo.spring.context.Test$B@4fb64261}
            System.out.println(l);
        }
    }

    public static class B {
    }
}
```


## props
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="props">
      <props>
        <prop key="admin">admin@example.org</prop>
        <prop key="support">support@example.org</prop>
      </props>
    </property>
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Properties;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setProps(Properties p) {
            // {admin=admin@example.org, support=support@example.org}
            System.out.println(p);
        }
    }

    public static class B {
    }
}
```


注意 Properties 值必须是 String 类型。因此 Spring 的 `<prop>` 不允许其它属性或子元素。参见 spring-beans-4.3.9.RELEASE.jar!/org/springframework/beans/factory/xml/spring-beans-4.3.xsd 第 1027 行。


## Collection merging
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="parent" abstract="true">
    <property name="props">
      <props>
        <prop key="admin">admin@example.org</prop>
        <prop key="support">support@example.org</prop>
      </props>
    </property>
  </bean>
  <bean id="a" class="org.example.demo.spring.context.Test.A" parent="parent">
    <property name="props">
      <props merge="true">
        <prop key="sales">admin@example.org</prop>
        <prop key="support">support@example.org.cn</prop>
      </props>
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Properties;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setProps(Properties p) {
            // {admin=admin@example.org, support=support@example.org.cn, sales=admin@example.org}
            System.out.println(p);
        }
    }
}
```


## Strongly-typed collection
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="map">
      <map>
        <entry key="one" value="9.99" />
        <entry key="two" value="2.75" />
        <entry key="six" value="3.99" />
      </map>
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Map;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setMap(Map<String, Float> m) {
            // {one=9.99, two=2.75, six=3.99}
            System.out.println(m);
        }
    }
}
```


# Null and empty string values
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="f1" value="" />
    <property name="f2">
      <value />
    </property>
    <property name="f3">
      <null />
    </property>
  </bean>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setF1(String s) {
            System.out.println("".equals(s)); // true
        }

        public void setF2(String s) {
            System.out.println("".equals(s)); // true
        }

        public void setF3(String s) {
            System.out.println(s == null); // true
        }
    }
}
```


# XML shortcut with the p-namespace
p, c, util 命名空间的支持参见 BeanDefinitionParserDelegate 第 1404 行
```java
	public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
		String namespaceUri = getNamespaceURI(ele);
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}
```


NamespaceHandlerResolver 创建在 XmlBeanDefinitionReader 第 545 行
```java
	protected NamespaceHandlerResolver createDefaultNamespaceHandlerResolver() {
		return new DefaultNamespaceHandlerResolver(getResourceLoader().getClassLoader());
	}
```


DefaultNamespaceHandlerResolver 第 88 行
```java
	public DefaultNamespaceHandlerResolver(ClassLoader classLoader) {
		this(classLoader, DEFAULT_HANDLER_MAPPINGS_LOCATION);
	}
```


而 DEFAULT_HANDLER_MAPPINGS_LOCATION 值为 META-INF/spring.handlers ，当前 spring-beans-4.3.9.RELEASE.jar!/META-INF/spring.handlers 文件内容如下：
```
http\://www.springframework.org/schema/c=org.springframework.beans.factory.xml.SimpleConstructorNamespaceHandler
http\://www.springframework.org/schema/p=org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler
http\://www.springframework.org/schema/util=org.springframework.beans.factory.xml.UtilNamespaceHandler
```


所以 p, c, util 命名空间分别由 META-INF/spring.handlers 中定义的三个 NamespaceHandler 处理。


例子


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" p:a="some string" p:b-ref="b" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setA(String a) {
            // some string
            System.out.println(a);
        }

        public void setB(B b) {
            // org.example.demo.spring.context.Test$B@396e2f39
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```


注意， p 命名空间不如标准 XML 格式灵活，比如，如果属性名就是以 ref 结尾，则看起来会比较迷惑（不会出错，因为 java 标识符不允许 "-" 字符）。


# XML shortcut with the c-namespace
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" c:a="some string" c:b-ref="b" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public A(String a, B b) {
            // some string
            System.out.println(a);
            // org.example.demo.spring.context.Test$B@13eb8acf
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```


where the constructor argument names are not available (usually if the bytecode was compiled without debugging information), one can use fallback to the argument indexes:


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" c:_0="some string" c:_1-ref="b" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public A(String a, B b) {
            // some string
            System.out.println(a);
            // org.example.demo.spring.context.Test$B@23fe1d71
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```


Due to the XML grammar, the index notation requires the presence of the leading _ as XML attribute names cannot start with a number


# Compound property names
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A">
    <property name="b.s" value="sss" />
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        private B b = new B();

        public B getB() {
            return b;
        }
    }

    public static class B {
        public void setS(String s) {
            // sss
            System.out.println(s);
        }
    }
}
```


如果 Test 第 12 行改为 `private B b;` 即不再初始化，则将抛异常 `org.springframework.beans.NullValueInNestedPathException: Invalid property 'b' of bean class [org.example.demo.spring.context.Test$A]: Value of nested property 'b' is null`

