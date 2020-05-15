---
title: log and jsr
abbrlink: 413131231
date: 2020-1-18 10:43:39
tag: 
    - log
category: java
---


## Log4j

 Log4j有7种不同的log级别，按照等级从低到高依次为：TRACE、DEBUG、INFO、WARN、ERROR、FATAL、OFF。如果配置为OFF级别，表示关闭log .

 log4j也很久没有更新了，现在已经有很多其他的日志框架对Log4j进行了改良，比如说SLF4J、Logback等 

## Log4j2

1. 插件式结构。Log4j 2支持插件式结构。我们可以根据自己的需要自行扩展Log4j 2. 我们可以实现自己的appender、logger、filter。
　　2. 配置文件优化。在配置文件中可以引用属性，还可以直接替代或传递到组件。而且支持json格式的配置文件。不像其他的日志框架，它在重新配置的时候不会丢失之前的日志文件。
　　3. Java 5的并发性。Log4j 2利用Java 5中的并发特性支持，尽可能地执行最低层次的加锁。解决了在log4j 1.x中存留的死锁的问题。
  4. 异步logger。Log4j 2是基于LMAX Disruptor库的。在多线程的场景下，和已有的日志框架相比，异步的logger拥有10倍左右的效率提升。

## logback

springboot 默认实现

## **slf4j**

一个统一的接口

> SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，而是通过Facade Pattern提供一些Java logging API，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。作者创建SLF4J的目的是为了替代Jakarta Commons-Logging。
> 实际上，SLF4J所提供的核心API是一些接口以及一个LoggerFactory的工厂类。在使用SLF4J的时候，不需要在代码中或配置文件中指定你打算使用那个具体的日志系统。类似于Apache Common-Logging，SLF4J是对不同日志框架提供的一个门面封装，可以在部署的时候不修改任何配置即可接入一种日志实现方案。但是，他在编译时静态绑定真正的Log库。使用SLF4J时，如果你需要使用某一种日志实现，那么你必须选择正确的SLF4J的jar包的集合（各种桥接包）。SLF4J提供了统一的记录日志的接口，只要按照其提供的方法记录即可，最终日志的格式、记录级别、输出方式等通过具体日志系统的配置来实现，因此可以在应用中灵活切换日志系统。

# jsr330 jsr250

### 注解：

### javax.annotation下：

- @Generated 标记这段代码不是手敲的，是自动生成的
- @ManagedBean 标记这个pojo已经被管理 jsf中定义
- @PostConstruct ： 标注bean的初始化工作函数
- @PreDestroy ： 标注bean的销毁工作函数
- @Priority ：标记在类或参数上，表示这个类被注入时的优先级
- @Resource 大多数情况下等价于spring内置的注解@Autowired 表示应用需要这个资源
- @Resources 只能标记与类上，需要多个资源

### javax.inject下

- @Inject 类似@Resource @Resource是java中JSR250中的规范，@Inject是java中JSR330中的规范
- @Named 类似@ManagedBean cdi 中定义 具体不太懂
- @Qualifier 自动注入的时候，使用了这个注解，那么在候选bean中，就会用这个注解指定的那个bean，作用在字段，方法，类，参数，注解上
- @Scope 表作用域
- @Singleton  继承 @Scope 表示ioc容器内只创建一次，且生命伴随整个ioc容器