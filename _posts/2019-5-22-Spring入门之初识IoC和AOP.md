---
layout:       post
title:        "Spring入门之初识IoC和AOP"
subtitle:     "SpringFramework的核心特性"
date:         2019-5-22 22：00
catalog:      true
tags:
    - Spring   
---

# Bean

## 配置
配置好的bean可以直接通过spring的**上下文**获取。
而配置只用告诉spring，一个**xml文件**的路径，spring就会基于这个xml生成一个对应的`context`对象。
类似 `Bean b = context.getBean("Bean.id")`.

### 配置项

- id **唯一标识**

- class 具体要实例化的类

- scope	**范围/作用域**

1. singleton	**一个Bean容器只存在一份**，但是可以基于一份xml文件生成两个context，这两个context容器可以有**各自的单例对象**
2. prototype	每次请求/使用时创建**新的实例**，destroy不生效，自动回收
3. request	**每次http请求**创建一个实例且只在这个**request**有效
4. session
5. global session	基于portlet（连接多个系统，例如OA、HR系统）的web应用中有效
- constructor arg 构造器参数

- properties	属性

- Autowiring mode	自动装配的方式

- lazy-initialization mode

- initialization/destruction method

### XML方式
**spring-ioc.xml**
```xml
<beans>
	<bean id="oneInterface" 
		class="com.tuo.interface.OneInterfaceImpl"></bean>
</beans>
```

### Java Config/注解 方式

扫描的策略需要在xml中配置。 可以指定扫描的包，还可以添加一些过滤器，实现更细致的扫描策略。
以下四个注解去注解的类都会被扫描并发现为Bean并注册到IoC容器中。

```xml
<context:component-scan base-package="com.tuo.bean"/>
```

1. @Component	元注解
用元注解去注解一个注解，这个注解也可以被自动发现
通用注解，可作用于任何bean

2. @Repository
通常用于注解DAO类，即持久层

3. @Service
通常用于注解Service类，即服务层

4. @Controller
通常用于注解Controller类，即控制层(mvc)

@Scope可以注解一个bean指定它的作用域，默认为`singleton`

还有一些其他的注解可以看源码学习， `@Configuration` `@Bean` `@Import` `@DependsOn`。

## 生命周期
1. 定义

2. 初始化
全局：

```xml
<beans 	...	...	...
	default-init-method="init" default-destroy-method="destroy">

</beans>
```

具体类：
- 实现`org.springframework.beans.factory.InitializingBean`接口， 覆盖`afterPropertisSet`方法
- 配置`init-method`方法

3. 使用

4. 销毁
 
- 实现`org.springframework.beans.factory.DisposableBean`接口， 覆盖`destory`方法
- 配置`destory-method`方法


## 自动装配

自动装配相较于普通的**设值注入和构造注入**，xml文件的bean配置更为简单，bean定义中也不需要写set方法/构造方法。
只需在bean中配置`Autowiring mode`, 或者配置全局的`default-autowire`.

1. NO

默认方式，不采取任何检查

2. byName

检查容器中id与这个bean属性完全一致的bean,并将其与属性自动装配。

3. byType

检查容器中与这个bean属性类型相同的bean。
- 如果只有一个则与之装配。
- 如果有多个，则抛出异常。
- 如果没有找到，则无事发生。

4. Constructor

## Resources&ResourceLoader

### Resource封装类型

- URLResource
- ClassPathResource
- FileSystemResource
- ServletContextResource
- InputStreamResource
- ByteArrayResource

### ResourceLoader

它是一个**接口**。

```java
public interface ResourceLoader{
	Resource getResource(String location);
}
```
`spring`中所有`application contexts`都实现了这个接口。
都可以当成`ResourceLoader`来用。
都可以获取到`Resource`的实例。

前缀：
- classpath:
- file:
- http:
- (none) 依赖于Application Context的获取方式

## Autowired等常用注解说明

- @Required 适用于setter方法上，仅仅表示这个属性必须在配置的时候填充一个确定值，可以是bean 标签的配置，也可以是自动装配。

- @Autowired 适用于setter方法、构造方法、和成员变量。与`@Requred`效果类似。

	默认情况下，如果找不到合适的bean来注入属性，将会导致autowiring失败并抛出异常。可以通过`@Autowired(required=false)`来避免.

	每个类只有一个构造器可以被注解为`@Autowired(required=true)`.

	`@Autowired`的必要属性建议使用传统的`@Required`.

	常用于注解那些众所周知的解析依赖性接口，比如：`BeanFactory`, `ApplicationContext`, `ResourceLoader`

	还可以实现数组或Map的自动注入。

# Bean容器context

## 基础
1. 包依赖：

	- `org.springframework.beans`
	- `org.springframework.context`

2. 核心类：

	- `BeanFactory`,提供配置结构和基本功能，加载并初始化Bean
	- `ApplicationContext`,保存了Bean对象并在Spring中广泛使用

## Bean容器的初始化方式

- **本地文件**

`FileSystemXmlAppliactionContext context = new FileSystemXmlAppliactionContext("F:xx/xx.xml")`
- **Classpath**

`ClasspathXmlAppliactionContext context = new ClasspathXmlAppliactionContext("classpath:spring-context.xml")`

- **Web应用中依赖servlet或listener**

`org.springframework.web.context.ContextLoaderListener/Servlet`



# IoC/DI

当需要一个对象时，不自己去创建这个对象，而是向spring**申请**，从**IoC容器**中拿一个出来用。

**实现IoC的方式是依赖注入。**

## 注入
1. 设值注入

```xml
<property name="injectionDAO" ref="injectionDAO"/>
```

通过调用**类的set方法**注入。

2. 构造注入

```xml
<constructor-arg name="injectionDAO" ref="injectionDAO"/>
```

通过调用类的**构造函数**注入。

# AOP

通过**预编译方式**和**运行期动态代理**实现程序功能的**统一维护**的技术。

> 预编译： *AspectJ*
> 动态代理：*JDK动态代理*、*CGLib*


## 主要功能 

1. 日志记录
2. 性能统计
3. 安全控制
4. 事务处理
5. 异常处理

## 相关概念

- 切面 Aspect：一个**关注点**的模块化，可能会**横切**多个对象

- 连接点 Joinpoint : 程序执行过程中的某个特定的点

- 通知 Advice ： 在切面的某个特定的连接点执行的**动作**

- 切入点 Pointcut: **匹配连接点的断言**，在AOP中通知和一个切入点表达式关联

- 引入 Introduction：在**不修改类代码**的前提下，为类**加新的方法和属性**。可能是借助预编译时修改字节码，如AspectJ

- 目标对象 Target Object, 被一个或多个切面**所通知的对象**。

- AOP代理 AOP Proxy:**AOP框架**创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）

- 织入 Weaving： 把切面**连接**到其他的应用程序类型或者对象上，并创建一个被通知的对象，分为：*编译时织入、类加载时织入、执行时织入*

## 通俗理解

![aop_concept](/img/spring/aop_concept.png)

AOP主要提供的都是一些和**核心业务功能**关联并不大的**辅助功能**，拿日志记录一点来说，只需在适当的位置前/后插入记录日志相关的代码。那么每个需要日志记录的关键点，也就是AOP里的**切入点Pointcut**， 都需要**前后插入这些代码**，也就是执行相关的动作（**通知 Advice**）。这将是很大的工作量。那么我们可以将这些和核心功能关联不大的功能，**单独做成模块**，这就是一个**切面**。**切面**是我们开发中同样重要的、**垂直于核心功能**的一个**关注点**。那么我们的切面准备好了以后，如何应用于核心业务功能呢? 这就需要用到Spring AOP提供框架来进行合适的**引入**。主要有以下几种引入方式：

- ***AspectJ*** 修改字节码
- ***CGLib*** 创建类的子类，也就是装饰类, 覆写需要通知的类
- ***JDK 动态代理*** 基于反射的JAVA API，提供动态代理的相关框架与接口， 只能代理接口和接口集

## Spring AOP

### 代理模式

1. 远程代理
2. 动态代理
3. 静态代理

### AspectJ

**编译期**的AOP，**检查**代码并**匹配**连接点与切入点的代价是高昂的，应当尽量定义良好的切入点。

### JDK 动态代理

Spring AOP 默认的代理模式，使用JAVA API，使得任何接口（或者接口集）都可以被代理。

### CGLib 代理

如果一个业务对象并没有实现一个接口时使用。

原理：
**运行时生成目标类的子类**，Spring 配置这个生成的子类委托方法调用到原来的目标。体现了**装饰模式**，子类来**织入通知**。
既然是**继承**，那么类的`final`方法不能被**通知**。因为它们不能被**覆盖**。