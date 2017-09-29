---
title: Spring ioc学习03
date: 2017-09-28 16:31:20
tags: java
---

#### 在Bean的整个生命周期上配置初始化和destory方法

有时候我们可能希望在一个bean对象的初始化和结束的时候调用特定的方法

Spring提供了几种机制进行支持

- @PostConstruct 注解的形式
- 实现 InitializingBean 接口
- 在xml上配置init方法 
  ```xml
  <bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
  ```

如果一个对象定义如上3种方式都使用了， 那么执行的顺序是按照上面的顺序方式进行执行，既先执行@PostConstruct的方法、再实现InitializingBean 接口的方法、最后调用xml上的init方法


destory的流程和初始化相似 

Destroy methods are called in the same order:

- Methods annotated with @PreDestroy
- destroy() as defined by the DisposableBean callback interface
- A custom configured destroy() method