---
title: Spring ioc学习02
date: 2017-09-28 16:31:15
tags: java
---

### 继续学习Ioc Container中配置细节和依赖

#### 1. p-namespace的使用

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```
<!--more-->

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

使用 xmlns:p 定义了p命名空间，大大简化了 bean的xml配置方式

---

#### 2.初始化集合类属性

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

> In the <list/>, <set/>, <map/>, and <props/> elements, you set the properties and arguments of the Java Collection types List, Set, Map, and Properties, respectively.

---

#### 3. 复合属性定义

```xml
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

这段定义中 定义了 Bar中的 fred属性的bob属性的sammy被设置成了123

如果fred 是Null 或者 bob是 Null 将会报出一个NullPointerException

--- 

#### 4. depense-on 参数

通常定义bean之间的依赖关系是通过ref关键字.但是有些特殊情况需要使用depense-on来声明依赖关系，也可以说是声明了 不同bean的初始化时机. 比如下面这个例子中,在初始化生成ExampleBean之前，先初始化了ManagerBean，
同时，在销毁ExampleBean之前，先销毁了ManagerBean.

**所以，depense-on不但控制着初始化顺序，还控制着析构顺序**

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

如果感到疑惑可以参考 [depense-on参考例子](http://yanln.iteye.com/blog/2210723)

---