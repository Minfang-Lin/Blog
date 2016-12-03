---
title: Spring学习笔记（一）
date: 2016-10-08 10:00
tags: [Spring,JavaWeb]
---

## Spring框架是什么？

The Spring Framework is a lightweight solution and a potential one-stop-shop for building your enterprise ready applications.

## Srping framwork的核心技术

Inversion of Control 控制反转
Aspect-Oriented Programming 面向切面编程

## 控制反转与依赖注入

使程序组件或类之间尽量形成一种松耦合的结构，开发人员在使用类的实例之前需要创建对象的实例。IoC将创建实例的任务交给IoC容器，这样开发应用代码只需要直接使用类的实例，这就是IoC控制反转。通常用一个所谓的好莱坞原则（Don't call me. I will call you.）来比喻这种控制反转关系。Martin Fowler曾专门写了一篇文章“Inversion of Control Containers and the Dependency Injection pattern”讨论控制反转这个概念，并提出一个更为准确的概念，即“依赖注入”。

<!-- more -->

## Ioc容器初始化

ApplicationContext是一个IoC容器，他扩展了BeanFactory容器并添加了对I18N和生命周期事件和发布监听等强大的功能。ApplicationContext接口有如下3个实现类，可以实例化其中任何一个类来创建Spring的ApplicationContext容器。

1. 在web应用场景下，可以通过WebApplicationContext初始化ApplicationContext：
``` xml
<!-- web.xml -->
  <context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:application-context.xml</param-value>
  </context-param>
  <listener>
  	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
  </listener>
```

2. ClassPathXmlApplicationContext类
从当前路径中检索配置文件并加载来创建容器的实例，语法格式如下：
``` java
ApplicationContext context = 
	new ClassPathXmlApplicationContext("application-context.xml");
```

3. FileSystemXmlApplicationContext类
该类不从路径中获取配置文件，而是通过参数指定配置文件的位置，它可以获取类路径之外的资源。
``` java
ApplicationContext context = 
	new FileSystemXmlApplicationContext("application-context.xml");
```

## Bean定义

``` xml
<!-- application-context.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="screwDriver" class="com.netease.course.ScrewDriver"></bean>
</beans>
```

`id`表示bean的唯一标识符，`class`表示这个bean所依赖的类名.

## Bean使用

``` java
// 初始化容器
ApplicationContext context = new ClassPathXmlApplicationContext("application-context.xml");
// 获取对象
ScrewDriver screwDriver = context.getBean("screwDriver", ScrewDriver.class);	
// 使用对象
screwDriver.use();
```

## Bean的作用域

* singleton：默认为singleton，但也可以通过`scope="singleton`显式定义。
* prototype：每次引用创建一个新的实例，通过`scope="prototype`定义。
* request
* session
* global session
* application

## Bean生命周期回调

``` java
public interface InitializingBean {
	void afterPropertiesSet() throws Exception;
}

public interface DisposableBean {
	void destory() throws Exception;
}
```

更简单的方法：

``` xml
<bean id="screwDriver" class="com.netease.course.ScrewDriver"
		init-method="init" destroy-method="cleanup"></bean>
```

``` java
public class ScrewDriver {
	public void init() {
		System.out.println("Init screwdriver");
	}
	public void cleanup() {
		System.out.println("Cleanup screwdriver");
	}
}
```

测试代码：

``` java
public class TestContainer {

	public static void main(String[] args) {
		
		// 初始化容器
		ApplicationContext context = 
			new ClassPathXmlApplicationContext("application-context.xml");
		// 获取对象
		ScrewDriver screwDriver = context.getBean("screwDriver", ScrewDriver.class);	
		// 使用对象
		screwDriver.use();
		
		((ConfigurableApplicationContext)context).close();
	}

}
```

## 依赖注入方式

#### 通过构造函数：主要应用于强依赖

螺丝刀刀头接口：
``` java
// Header.java
package com.netease.course;

public interface Header {
	public void doWork();
	public String getInfo();
}
```

一字螺丝刀刀头实现：
``` java
// StraightHeader.java
package com.netease.course;

public class StraightHeader implements Header {

	private String color;
	private int size;

	public StraightHeader(String color, int size) {
		this.color = color;
		this.size = size;
	}

	@Override
	public void doWork() {
		System.out.println("Do work with straight header");
	}

	@Override
	public String getInfo() {
		return "StraightHeader color:" + color + ", size: " + size;
	}

}
```

配置文件：
``` xml
<!-- application-context.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd">

	<bean id="header" class="com.netease.course.StraightHeader">
		<constructor-arg value="red"></constructor-arg> 
		<constructor-arg value="15"></constructor-arg>
	</bean>
</beans>
```

`constructor-arg`也可通过`index=0`设置参数对应构造函数参数列表顺序，`name="color"`设置对应构造函数参数表参数名称，`type="java.lang.String"`设置参数数据类型。


测试代码:
``` java
package com.netease.course;

import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestContainer {

	public static void main(String[] args) {
		
		ApplicationContext context = 
			new ClassPathXmlApplicationContext("application-context.xml");
	
		Header header = context.getBean("header", StraightHeader.class);
		System.out.println(header.getInfo());
		header.doWork();
		
		((ConfigurableApplicationContext)context).close();
	}

}
```

* 通过构造函数集合的方式：

`StraightHeader.java`中添加一个新的构造函数：

``` java
public StraightHeader(Map<String, String> paras) {
	this.color = paras.get("color");
	this.size = Integer.valueOf(paras.get("size"));
}
```

`application-context.xml`修改为：
``` xml
<constructor-arg>
	<map>
		<entry key="color" value="red"></entry>
		<entry key="size" value="14"></entry>
	</map>
</constructor-arg>
```

* 通过配置项加载的方式：

`StraightHeader.java`中添加一个新的构造函数：

``` java
public StraightHeader(Properties props) {
	this.color = props.getProperty("color");
	this.size = Integer.valueOf(props.getProperty("size"));
}
```

`application-context.xml`修改为：
``` xml
<constructor-arg>
	<props>
		<prop key="color">green</prop>
		<prop key="size">16</prop>
	</props>
</constructor-arg>
```

* 通过配置文件的方式加载参数值：

`application-context.xml`修改为：

``` xml
<bean id="header" class="com.netease.course.StraightHeader">
	<constructor-arg name="color" value="${color}"></constructor-arg> 
	<constructor-arg name="size" value="${size}"></constructor-arg>
</bean>

<bean id="headerProperties" 
class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
<property name="location" value="classpath:header.properties"></property>
</bean>
```
并在`src\main\resources`目录下创建一个`header.properties`配置文件。

* 通过构造函数的方式将一个Bean注入到另一个Bean中：

``` java
// ScrewDriver.java
package com.netease.course;

public class ScrewDriver {
	
	private Header header;

	public ScrewDriver(Header header) {
		this.header = header;
	}

	public void use() {
		System.out.println("Use header: " + header.getInfo());
		header.doWork();
	}
}

```

``` xml
<!-- application-context.xml -->
<bean id="screwDriver" class="com.netease.course.ScrewDriver">
	<constructor-arg>
		<ref bean="header" />
	</constructor-arg>
</bean>
```


``` java
// Test
package com.netease.course;

import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestContainer {

	public static void main(String[] args) {
		
		ApplicationContext context = 
			new ClassPathXmlApplicationContext("application-context.xml");
		
		ScrewDriver screwDriver = context.getBean(ScrewDriver.class);
		screwDriver.use();
		
		((ConfigurableApplicationContext)context).close();
	}

}
```

#### Setter方法：主要应用于可选依赖

``` xml
<!-- application-context.xml -->
<bean id="header" class="com.netease.course.StraightHeader">
	<property name="color">
		<value>red</value>
	</property>
	<property name="size">
		<value>11</value>
	</property>
</bean>
```
StraightHeader类需要有Setter和Getter方法，以及默认无参构造函数。

## 自动装配

* byName：根据Bean名称
* byType：根据Bean类型
* constructor：构造函数，根据类型

#### 通过byName自动装配

在xml配置文件中修改：

``` xml
<bean id="screwDriver" class="com.netease.course.ScrewDriver" autowire="byName">
</bean>
```

并在ScrewDriver类中定义Setter方法。

#### 通过constructor自动装配

在xml配置文件中修改：

``` xml
<bean id="screwDriver" class="com.netease.course.ScrewDriver" autowire="constructor">
</bean>
```

并在ScrewDriver类中定义一个带所有参数的构造函数。

## Annotation

* `@Component`：定义Bean
* `@Value`：properties注入
* `@Autowired` & `@Resource`：自动装配依赖
* `@PostConstruct` & `@PreDestory`：生命周期回调

如果需要使用Annotation，在xml中要添加一个配置：
``` xml
<context:component-scan base-package="com.netease.course" />
```
package代表需要搜索的包名。

使用`@Component`和`@Value`：

``` java
// StraightHeader.java
package com.netease.course;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component("header")
public class StraightHeader implements Header {
	@Value("${color}")
	private String color;
	@Value("${size}")
	private int size;

	@Override
	public void doWork() {
		System.out.println("Do work with straight header");
	}

	@Override
	public String getInfo() {
		return "StraightHeader color:" + color + ", size: " + size;
	}

}
```

使用`@Autowired`自动装配：

``` java
// ScrewDriver.java
package com.netease.course;

import org.springframework.beans.factory.annotation.Autowired;

public class ScrewDriver {
	@Autowired
	private Header header;

	public void use() {
		System.out.println("Use header: " + header.getInfo());
		header.doWork();
	}
}
```

## AOP术语

* Aspect：日志、安全等功能
* Join point：函数执行或者属性访问
* Advice：在某个函数执行点上要执行的切面功能
* Pointcut：匹配横切目标函数的表达式

#### Advice类型

* Before：函数执行之前
* After returning：函数正常返回之后
* After throwing：函数抛出异常之后
* After finally：函数返回之后
* Around：函数执行前后

#### Spring AOP

* @AspectJ annotation-based AOP 需要添加aspectjweaver.jar
* XML schema-based AOP

#### @AspectJ AOP

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop-2.0.xsd">

	<aop:aspectj-autoproxy />

</beans>
```

#### 定义Aspect

``` xml
<bean id="loggingAspect" class="com.netease.course.LoggingAspect">
	<!-- configure properties of aspect here as normal -->
</bean>
```

``` java
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class LoggingAspect {

}
```

#### 定义Pointcut

``` java
@Pointcut("execution(* com.netease.course.Caculator.*(..))")
private void arithmetic() {}
```

* Pointcut表达式
``` bash
designator(modifiers? return-type declaring-type? name(param) throws?)
```

1. designator: execution代表函数执行过程，within代表在某个包或某个类下运行的函数。
2. modifiers：public,private...
3. return-type：返回类型，用`*`表示所有返回类型
4. declaring-type：申明类型，包名或是类名
5. name：函数名，可以用`*`匹配所有函数，或者可以用正则表达式形式
6. param：参数列表，`()`表示匹配无参函数，`(..)`表示匹配任意参数个数的函数
7. throws：异常类型

* Pointcut示例

    所有public函数
    ``` java
    execution( public * *(..))
    private void publicMethod() {}
    ```

    所有DAO模块中的public函数
    ``` java
    execution( public * com.netease.dao.*.*(..))
    private void publicDaoMethod() {}
    ```

    所有以save开头的函数
    ``` java
    execution( * save*(..))
    private void saveMethod() {}
    ```

    所有以save开头的public函数
    ``` java
    execution(publicMethod() && saveMethod())
    private void publicSaveMethod() {}
    ```

#### 定义Advice

``` java
// 以pointcut表达式名称的方式
@Before("com.netease.course.LoggingAspect.arithmetic()")
public void doLog() {
	//
}
```

``` java
// 以pointcut表达式的方式
@Before("execution(* com.netease.course.Caculator.*(..))")
public void doLog() {
	//
}
```

``` java
@AfterReturning("com.netease.course.LoggingAspect.arithmetic()")
public void doLog() {
	//
}
```

``` java
@AfterThrowing("com.netease.course.LoggingAspect.arithmetic()")
public void doLog() {
	//
}
```

``` java
@After("com.netease.course.LoggingAspect.arithmetic()")
public void doLog() {
	//
}
```

获取函数上下文信息：

``` java
@Before("com.netease.course.LoggingAspect.arithmetic()")
public void doLog(JoinPoint jp) {
	System.out.println(jp.getSignature() + "," + jp.getArgs());
}
```

Around获取上下文信息时，需要JoinPoint的子类ProceedingJoinPoint

``` java
@Around("com.netease.course.LoggingAspect.arithmetic()")
public Object doLog(ProceedingJoinPoint pjp) {
	System.out.println("start method: " + pjp.toString());
	Object retVal = pjp.proceed();
	System.out.println("stop method: " + pjp.toString());
	return retVal;
}
```

取得函数返回值

``` java
@AfterReturning(
	pointcut="com.netease.course.LoggingAspect.arithmetic()",
	returning="retVal")
public void doLog(Object retVal) {
	// do something with retVal
}
```

取得抛出的异常

``` java
@AfterThrowing(
	pointcut="com.netease.course.LoggingAspect.arithmetic()",
	throwing="ex")
public void doLog(IllegalArgumentException ex) {
	// do something with ex
}
```

获取目标函数参数

``` java
@Before("com.netease.course.LoggingAspect.arithmetic() && args(a, ..)")
public void doLog(JoinPoint jp, int a) {
	// do something with parameters
}
```