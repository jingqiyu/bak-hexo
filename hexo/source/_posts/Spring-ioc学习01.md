---
title: Spring ioc学习01
date: 2017-09-28 16:25:36
tags: java
---


## 学习Srping Ioc ##

The **org.springframework.beans** and **org.springframework.context** packages are the basis for Spring Framework’s IoC container.

官网的一段说明

This chapter covers the Spring Framework implementation of the Inversion of Control (IoC) [1] principle. IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies, that is, the other objects they work with, only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse, hence the name Inversion of Control (IoC), of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes, or a mechanism such as the Service Locator pattern.

翻译过来其实就是原来我们在某一类里面依赖了一些外部的某些类或者接口的实现, 是由我们类自身控制着到底选择哪一个实现或者类。现在将控制权利反转，就是IOC，而本质上DI依赖注入是实现这一目标的一种技术实现方式.

BeanFactory接口定义了一些高级配置，可以管理任何类型的对象. 而ApplicationContext是BeanFactory的子接口 提供了一些AOP\和特定上下文的支持（ApplicationContext) 简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext添加了更多的特定于企业的功能。在Spring中，构成应用程序主干的对象，以及由Spring IoC容器管理的对象称为bean。bean是一个被Spring IoC容器实例化、组装和管理的对象。


**ApplicationContext** 代表了Spring IoC容器，负责实例化、配置和组装上述的bean

ApplicationContext通过元数据的配置来填充到Container中 请看官网给出的一张图片理解

<!--more-->

![Container](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/container-magic.png)

下面介绍两种方式

- xml方式
- 注解的方式

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

实例化一个容器也很简单
```java
ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
```

services.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

daos.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

通常，每个XML配置文件都表示体系结构中的一个逻辑层或模块。

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

下面看看如何使用这个container

```java
applicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

Bean的使用

id 唯一标识了一个bean对象 命名规范参照bean的名称从小写字母开始，然后从那时起开始使用驼式。这样的名字的例子将是(没有引号)“accountManager”、“accountService”、“userDao”、“loginController”等等。

起别名的方式
```java
<alias name="fromName" alias="toName"/>
```

---


DI 的两种方式

- 基于构造函数的注入
- 基于Setter方式的注入
 
基于构造函数:
``` java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...

}
```

有一些情况 可能我们的构造函数里面是两个基础类型 比如 一个int，一个String，可以用以下两种方式解决
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```
或者
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
