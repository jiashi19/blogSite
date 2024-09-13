---
title: spring知识点
categories:
  - [java]
  - 秋招
date: 2024-07-21 20:46:35
tags:
password: 12345s
---

# Springboot 知识点总结

<!-- more -->

## IOC&DI

基础概念：

- 内聚：软件中各个功能模块内部的功能联系。
- 耦合：衡量软件中各个层/模块之间的依赖、关联的程度。
- 软件设计原则：高内聚低耦合。

**重要思想**：

- **控制反转：** Inversion Of Control，简称**IOC**。对象的创建控制权由程序自身转移到外部（容器），这种思想称为控制反转。
- **依赖注入：** Dependency Injection，简称**DI**。容器为应用程序提供运行时，所依赖的资源，称之为依赖注入。
- **Bean**对象：容器中创建、管理的对象，称之为bean。

## Spring 中的 @Component 注解

`@Component` 允许 Spring 自动检测自定义 Bean：

- 扫描应用，查找注解为 `@Component` 的类
- 将它们实例化，并注入任何指定的依赖
- 在需要的地方注入



`@Controller`、`@Service` 和 `@Repository`都是由`@Component`而来的（衍生）注解，基本作用相同。



`@ComponentScan`：Spring 使用 `@ComponentScan` 注解将它们收集到 `ApplicationContext` 中。Spring Boot 中的 `@SpringBootApplication` 注解就是一个包含了 `@ComponentScan` 的注解。只要 `@SpringBootApplication` 类位于项目根目录，它就会默认扫描应用中定义的每个 `@Component`。



**@Resource 与 @Autowired区别**：

- @Autowired 是spring框架提供的注解，而@Resource是JDK提供的注解。（存在多个类型的Bean可通过在有@Autowired的基础上加@Primary和@Qualifier指明单个bean）

- @Autowired 默认是**按照类型**（by TYPE）注入，而@Resource（by NAME）默认是**按照名称**注入。

  ```java
  @RestController
  public class EmpController {
  	@Resource(name = "empServiceB")
      private EmpService empService ;
   //...
   }
  ```

  

参考链接：

[Spring 中的 @Component 注解 - spring 中文网 (springdoc.cn)](https://springdoc.cn/spring-component-annotation/)

### bean作用域

- singleton 单例：默认容器启动时创建。@Lazy 注解延迟初始化到第一次使用。
- prototype 非单例

```java
@Scope("prototype")
```

### 第三方bean

- 在配置类 @Configuration中配置第三方bean（@Bean）；

- Spring的@Bean注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理。产生这个Bean对象的方法Spring只会调用一次，随后Spring将会将这个Bean对象放在IOC容器中；
- @Bean是方法级别上的注解

## 事务

事务的概念：一组操作的集合，不可分割，要么同时成功，要么都失败。

spring注解：@Transactional

**设置事务模式：**在处理事务时，默认的连接模式是自动提交（auto-commit），即每执行一条SQL语句，都会立即提交事务。但是，为了确保事务的原子性和一致性，应用程序可能会切换到手动提交模式。第二条日志信息说明了系统将这个连接切换到了手动提交模式，意味着在事务结束之前，需要显式调用`commit`方法来提交事务，否则可以调用`rollback`方法回滚事务。

**事务属性：**

- rollbackFor:  默认下，only RuntimeException才回滚，而此参数可控制出现什么异常时进行事务回滚。
- propagation: 事务传播行为，默认REQUIRED（有则加入无则创建新）、REQUIRES_NEW（总是创建新）

[实际场景：无论操作（操作1）成功与否，记录日志（操作2），即操作2在操作1中，这里操作2需设置REQUIRES_NEW 有新的事务，不会随同操作1的事务一同回滚] 

## AOP

[细说Spring——AOP详解（AOP概览）-CSDN博客](https://blog.csdn.net/q982151756/article/details/80513340)

[Spring AOP 详细深入讲解+代码示例-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1357215)

概念：Aspect Oriented Programming、面向切面编程，面向特定方法的编程。

实现：动态代理

场景：记录操作日志，权限控制，事务管理，校验，缓存

**优点**：松耦合，模块化，非侵入性，可重用性

**核心概念**：

- 连接点joinpoint
- 切入点pointCut   Pointcut的作用就是提供一组**规则**来匹配joinpoint, 给满足规则的 joinpoint 添加 Advice ， **则一组匹配规则**
- 增强（通知）advice     `advice` 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 `Joint point` 上的。
- 切面Aspect 可以看作是point cut + advice
- 目标对象target

![示意图](https://s2.loli.net/2024/05/30/XRZT5WEeqIvwilF.png)

### Advice

种类：前置@Before，后置@After，环绕@Around，异常@AfterThrowing，最终@AfterReturning

注意：

- @Around 环绕通知需要自己调用ProceedingJoinPoint.proceed()来**执行原始方法**；
- @Around的返回值须指定为Object，接收**原始方法返回值**

```java
@Around("execution(* com.itheima.service.*.*(..))") //切入点表达式
public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
    //1
    long begin=System.currentTimeMillis();
    //2
    Object result=joinPoint.proceed();
    //3
    long end=System.currentTimeMillis();
    log.info("{}方法花费:{}ms",joinPoint.getSignature(),end-begin);
    return result;
}
```

### pointCut

```
execution(访问修饰符 ?  返回值  包名.类名.?方法名(方法参数) throws  异常?)
```

？号表示可省略的部分：

-  访问修饰符
-  包名.类名
-  throws异常

示例：

```java
@AfterThrowing("execution(public void com.itheima.service.impl.DeptServiceImpl.delete(java.lang.Integer))")
```

### joinpoint

1. 对于@Around，获取连接点信息只能使用ProceedingJoinPoint；
2. 对于其他通知，获取连接点信息只能用JoinPoint。



## springboot原理

### 起步依赖

based on：maven依赖传递

### 自动配置

spring容器启动后就会将一些配置类，bean对象**自动存入IOC**容器，不需要手动声明。

@ComponentScan （扫描）   **or** @Import （导入）

@EnableXXX 封装@ Import 注解

#### 源码分析

[SpringBoot 自动配置 (tencent.com)](https://cloud.tencent.com/developer/article/2120900)

```
@SpringBootApplication
-->
@SpringBootConfiguration
@EnableAutoConfiguration
```

## springboot设计模式
### IoC 与 DI
DI依赖注入是实现IoC的一种设计模式
### 工厂设计模式
Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象
### 单例设计模式
spring中的bean的默认就是单例singleton

单bean遇到线程安全问题：在 Bean 中尽量避免定义可变的成员变量。在类中定义一个 ThreadLocal 成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐）。
### 代理设计模式
AOP
