AutowireCapableBeanFactory
- AUTOWIRE\_NO - 0
- AUTOWIRE\_BY\_NAME - 1
- AUTOWIRE\_BY\_TYPE - 2
- AUTOWIRE\_CONSTRUCTOR - 3
- AUTOWIRE_AUTODETECT - 4


# default
## default default
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <beans>
    <bean id="a" class="org.example.demo.spring.context.Test.A" />
  </beans>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        AbstractBeanDefinition a = (AbstractBeanDefinition) applicationContext.getBeanFactory().getBeanDefinition("a");
        System.out.println(a.getAutowireMode()); // 0
        applicationContext.close();
    }

    public static class A {
    }
}
```


## default byName
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
  default-autowire="byName">
  <beans>
    <bean id="a" class="org.example.demo.spring.context.Test.A" />
  </beans>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        AbstractBeanDefinition a = (AbstractBeanDefinition) applicationContext.getBeanFactory().getBeanDefinition("a");
        System.out.println(a.getAutowireMode()); // 1
        applicationContext.close();
    }

    public static class A {
    }
}
```


## default byType
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
  default-autowire="byName">
  <beans default-autowire="byType">
    <bean id="a" class="org.example.demo.spring.context.Test.A" />
  </beans>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        AbstractBeanDefinition a = (AbstractBeanDefinition) applicationContext.getBeanFactory().getBeanDefinition("a");
        System.out.println(a.getAutowireMode()); // 2
        applicationContext.close();
    }

    public static class A {
    }
}
```


## default constructor
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
  default-autowire="byName">
  <beans default-autowire="byType">
    <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
  </beans>
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        AbstractBeanDefinition a = (AbstractBeanDefinition) applicationContext.getBeanFactory().getBeanDefinition("a");
        System.out.println(a.getAutowireMode()); // 3
        applicationContext.close();
    }

    public static class A {
    }
}
```


相关源码在 BeanDefinitionParserDelegate 第 356 行
```java
		String autowire = root.getAttribute(DEFAULT_AUTOWIRE_ATTRIBUTE);
		if (DEFAULT_VALUE.equals(autowire)) {
			// Potentially inherited from outer <beans> sections, otherwise falling back to 'no'.
			autowire = (parentDefaults != null ? parentDefaults.getAutowire() : AUTOWIRE_NO_VALUE);
		}
		defaults.setAutowire(autowire);
```


这是设置 beans 的 default-autowire 的值：
- 如果 beans 元素上定义了 default-autowire 且不为 default ，则使用之，否则
- 如果有上层嵌套的 beans 元素，取用它的 default-autowire ，否则
- 使用 no 。


在第 602 行
```java
		String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
		bd.setAutowireMode(getAutowireMode(autowire));
```


在第 693 行
```java
	public int getAutowireMode(String attValue) {
		String att = attValue;
		if (DEFAULT_VALUE.equals(att)) {
			att = this.defaults.getAutowire();
		}
		int autowire = AbstractBeanDefinition.AUTOWIRE_NO;
		if (AUTOWIRE_BY_NAME_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_BY_NAME;
		}
		else if (AUTOWIRE_BY_TYPE_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_BY_TYPE;
		}
		else if (AUTOWIRE_CONSTRUCTOR_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR;
		}
		else if (AUTOWIRE_AUTODETECT_VALUE.equals(att)) {
			autowire = AbstractBeanDefinition.AUTOWIRE_AUTODETECT;
		}
		// Else leave default value.
		return autowire;
	}
```


这是设置 bean 的 autowire 的值：
- 如果属性值为 default ，取用上面计算的 beans 的最终 default-autowire
- 如果还是 default ，返回 AbstractBeanDefinition.AUTOWIRE_NO ，否则返回各自对应的值。


从 spring-beans-4.3.xsd 中查看到 default-autowire/autowire 的缺省值都是 default ，并且 xsd:documentation 中说 autowire 属性不会被继承(This attribute will not be inherited by child bean definitions)。


# byName
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byName" />
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
        public void setB(B b) {
            // in setB org.example.demo.spring.context.Test$B@32eebfca
            System.out.println("in setB " + b);
        }

        public void setC(B b) {
            System.out.println("in setC " + b);
        }
    }

    public static class B {
    }
}
```


结论
___

如果 bean a 有个属性 b ，则查找名为 b 的 bean
- 如果找到（不可能找到多个），设置 bean a 的属性 b 为 bean b
- 如果没有找到（比如属性 c ），什么都不做。


相关源码参见 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory 第 1288 行
```java
	protected void autowireByName(
			String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {

		String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);
		for (String propertyName : propertyNames) {
			if (containsBean(propertyName)) {
				Object bean = getBean(propertyName);
				pvs.add(propertyName, bean);
				registerDependentBean(propertyName, beanName);
				if (logger.isDebugEnabled()) {
					logger.debug("Added autowiring by name from bean name '" + beanName +
							"' via property '" + propertyName + "' to bean named '" + propertyName + "'");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Not autowiring property '" + propertyName + "' of bean '" + beanName +
							"' by name: no matching bean found");
				}
			}
		}
	}
```


# byType
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
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
        public void setB(B b) {
            // org.example.demo.spring.context.Test$B@c81cdd1
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```
___


如果修改 test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" />
</beans>
```


再次运行 Test 抛异常 `org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.example.demo.spring.context.Test$B' available: expected single matching bean but found 2: b,c`
___


再修改 test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" primary="true" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$B@42607a4f
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$B@c81cdd1
        System.out.println(applicationContext.getBean("c"));
        applicationContext.close();
    }

    public static class A {
        public void setB(B b) {
            // org.example.demo.spring.context.Test$B@c81cdd1
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```
___


再修改 test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" primary="true" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" primary="true" />
</beans>
```


运行 Test 抛异常 `org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.example.demo.spring.context.Test$B' available: more than one 'primary' bean found among candidates: [b, c]`
___


再次修改 test.xml ，显式定义依赖：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType">
    <property name="b" ref="b" />
  </bean>
  <bean id="b" class="org.example.demo.spring.context.Test.B" primary="true" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" primary="true" />
</beans>
```


```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$B@553a3d88
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$B@1fc2b765
        System.out.println(applicationContext.getBean("c"));
        applicationContext.close();
    }

    public static class A {
        public void setB(B b) {
            // org.example.demo.spring.context.Test$B@553a3d88
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```
___


再次修改 test.xml ，这次将 bean b, c 的定义都去掉
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
</beans>
```


运行正常，没有输出。
```java
package org.example.demo.spring.context;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        applicationContext.close();
    }

    public static class A {
        public void setB(B b) {
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```
___


结论：


如果 bean a 有个属性 b 的类为 B ，则查找类型为 B 的 bean
- 如果仅找到一个，设置 bean a 的属性 b 为找到的 bean
- 如果没有找到，什么都不做
- 如果找到多个
  - 如果其中只有一个 bean 的 primary 属性为 true ，返回它
  - 如果其中没有 primary 为 true 或者有多个为 true ，抛出异常


相关源码参见 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireByType() 方法。


注意，在 AbstractAutowireCapableBeanFactory.autowireByName/autowireByType 中都先调用了 unsatisfiedNonSimpleProperties 去得到没有显式配置依赖的属性，因此，如果显式配置了依赖，即使有多个 candidate 也不会报错（但是是一个潜在的危险，将来可能会有问题）。


# constructor
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
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
        public A(B b) {
            // org.example.demo.spring.context.Test$B@87f383f
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```
___


修改 test.xml 再添加一个类型为 B 的 bean
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" />
</beans>
```


仍然可以正常运行！
___


再修改 test.xml ，将 bean 的 id 改为别的名字（与构造函数中不一样）


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
  <bean id="d" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" />
</beans>
```


运行 Test 抛异常 `org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.example.demo.spring.context.Test$B' available: expected single matching bean but found 2: d,c`
___


添加 primary="true"
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
  <bean id="d" class="org.example.demo.spring.context.Test.B" primary="true" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" />
</beans>
```


又可以正常运行。
___


再添加 primary="true"
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
  <bean id="d" class="org.example.demo.spring.context.Test.B" primary="true" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" primary="true" />
</beans>
```


又抛异常 `org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'org.example.demo.spring.context.Test$B' available: more than one 'primary' bean found among candidates: [d, c]`
___


再次修改 test.xml ，显式定义依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor">
    <constructor-arg name="b" ref="c" />
  </bean>
  <bean id="d" class="org.example.demo.spring.context.Test.B" primary="true" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" primary="true" />
</beans>
```


运行正常。
___


再次修改 test.xml ，这次将 bean d, c 的定义都去掉
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="constructor" />
</beans>
```


运行抛异常 `org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'org.example.demo.spring.context.Test$B' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}`
___


结论：


constructor 与 byType 类似，但是应用于构造器参数。


如果 bean a 有个构造器参数 b 的类为 B ，则查找类型为 B 的 bean
- 如果仅找到一个，设置 bean a 的构造器参数 b 为找到的 bean
- 如果没有找到，**将抛异常**，这是与 byType 的不同之处
- 如果找到多个
  - **如果其中一个的 identifier 与构造器参数名相同，返回它**，这也是与 byType 的不同
  - 如果其中只有一个 bean 的 primary 属性为 true ，返回它
  - 如果其中没有 primary 为 true 或者有多个为 true ，抛出异常。


相关源码参见 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor() 方法。


# autodetect
参见 `org.springframework.beans.factory.support.AbstractBeanDefinition.getResolvedAutowireMode()`
```java
	public int getResolvedAutowireMode() {
		if (this.autowireMode == AUTOWIRE_AUTODETECT) {
			// Work out whether to apply setter autowiring or constructor autowiring.
			// If it has a no-arg constructor it's deemed to be setter autowiring,
			// otherwise we'll try constructor autowiring.
			Constructor<?>[] constructors = getBeanClass().getConstructors();
			for (Constructor<?> constructor : constructors) {
				if (constructor.getParameterTypes().length == 0) {
					return AUTOWIRE_BY_TYPE;
				}
			}
			return AUTOWIRE_CONSTRUCTOR;
		}
		else {
			return this.autowireMode;
		}
	}
```


可见如果为 autodetect ，如果有 public 的无参数构造器，则使用 byType ，否则使用 constructor 。


# wire arrays and typed-collections and strongly-typed Maps
src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byType" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" />
  <bean id="c" class="org.example.demo.spring.context.Test.B" />
</beans>
```


```java
package org.example.demo.spring.context;

import java.util.Arrays;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$B@306279ee
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$B@545997b1
        System.out.println(applicationContext.getBean("c"));
        applicationContext.close();
    }

    public static class A {
        public void setB(B[] bs) {
            // [org.example.demo.spring.context.Test$B@306279ee, org.example.demo.spring.context.Test$B@545997b1]
            System.out.println(Arrays.asList(bs));
        }
    }

    public static class B {
    }
}
```


将 Test 改为使用 List
```java
package org.example.demo.spring.context;

import java.util.List;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$B@5d11346a
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$B@7a36aefa
        System.out.println(applicationContext.getBean("c"));
        applicationContext.close();
    }

    public static class A {
        public void setB(List<B> bs) {
            // [org.example.demo.spring.context.Test$B@5d11346a, org.example.demo.spring.context.Test$B@7a36aefa]
            System.out.println(bs);
        }
    }

    public static class B {
    }
}
```


改为使用 Map
```java
package org.example.demo.spring.context;

import java.util.Map;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring/test.xml");
        // org.example.demo.spring.context.Test$B@6ec8211c
        System.out.println(applicationContext.getBean("b"));
        // org.example.demo.spring.context.Test$B@7276c8cd
        System.out.println(applicationContext.getBean("c"));
        applicationContext.close();
    }

    public static class A {
        public void setB(Map<String, B> map) {
            // {b=org.example.demo.spring.context.Test$B@6ec8211c, c=org.example.demo.spring.context.Test$B@7276c8cd}
            System.out.println(map);
        }
    }

    public static class B {
    }
}
```


# Excluding a bean from autowiring
autowire-candidate 对 byName autowire 不起作用


src/main/resources/spring/test.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="a" class="org.example.demo.spring.context.Test.A" autowire="byName" />
  <bean id="b" class="org.example.demo.spring.context.Test.B" autowire-candidate="false" />
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
        public void setB(B b) {
            // org.example.demo.spring.context.Test$B@32eebfca
            System.out.println(b);
        }
    }

    public static class B {
    }
}
```


