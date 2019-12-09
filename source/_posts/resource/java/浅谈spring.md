---
title: 浅谈spring
abbrlink: 16104523
date: 2019-11-28 10:43:39
tag: 
    - spring
    - java
category: java
---

## 老生长谈 IOC

学习spring 首先要祭上spring 官方那张图

![spring framework runtime](https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/images/spring-overview.png)

spring framework runtime

可以看出最核心的那个 是一个容器，IOC是spring最基本的能力，简单来说inversion of control，就是把软件的部分控制由程序员交给了spring ，最典型的就是对象的新建和初始化（依赖建立）的反转，除了解耦和减少程序员代码，spring的IOC还为其他的功能打下了基础。

### 粗略的组件理解

1. core 提供了IOC过程中需要的“工具”，所有操作都需要它

2. context 上下文，在计算机系统中，进程执行时有进程上下文，如果进程在执行的过程中遇到了中断，CPU 会从用户态切换为内核态（当然这个过程用户进程是感知不到的，由硬件来实现的），此时进程处于的进程上下文会被切换到中断上下文中，从而可以根据中断号去执行相应的中断程序。

   内容出处

   - DefaultListableBeanFactory
     这就是大家常说的 ioc 容器，它里面有很多 map、list。spring 帮我们创建的 singleton 类型的 bean 就存放在其中一个 map 中。我们定义的监听器（ApplicationListener）也被放到一个 Set 集合中
   - BeanDefinitionRegistry
     把一个 BeanDefinition 放到 beanDefinitionMap。
   - AnnotatedBeanDefinitionReader
     针对 AnnotationConfigApplicationContext 而言。一个 BeanDefinition 读取器。
   - 扩展点集合
     存放 spring 扩展点（主要是 BeanFactoryPostProcessor、BeanPostProcessor）接口的 list 集合。

3. Beans 存入到spring容器中的对象

#### AOP

- AOP 核心概念和IOC一样非常好理解，就是抽取各部分中不重要或者重复的部分，拿出来（从侧面切出来）将原本程序从上到下执行逻辑中不重要的分离到侧面（常见的记录日志，权限控制）

- AspectJ 是另一种 aop框架（貌似比spring AOP更强大？）原本的aop的编程方式太反人类了，所以spring后来借用了aspectj的方式，但核心还是spring

- spring借用cglib而不是jdk自己的动态代理主要是应为要完成基于类的代理而不是为了啥效率

  > （1）JDK动态代理只能对实现了接口的类生成代理，而不能针对类
  > （2）CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法
  > 因为是继承，所以该类或方法最好不要声明成final