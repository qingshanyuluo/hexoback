---
title: Mybatis学习
abbrlink: 16108
tag: 
    - mybatis
    - java
    - Dao
category: 后端框架
---
## Resources.getResourceAsStream(resource)源码探究:

```java
String resource = "config.xml";InputStream inputStream = Resources.getResourceAsStream(resource);
```

mybatis 的 getResourceAsStream(resource) 方法主要通过

```java
ClassLoader[] getClassLoaders(ClassLoader classLoader) {  return new ClassLoader[]{      classLoader,      defaultClassLoader,      Thread.currentThread().getContextClassLoader(),      getClass().getClassLoader(),      systemClassLoader};}
```

获得classloader 然后调用　getResourceAsStream(resource) 方法来获取classpath 下文件的InputStream

## 生命周期和域相关文档翻译

看完基本用法后发现对生命周期这一块理解薄弱在此欲小小翻译一下[官方文档中GettingStarted](https://mybatis.org/mybatis-3/getting-started.html)中关于生命周期的说法

> ### Scope and Lifecycle
>
> It’s very important to understand the various scopes and lifecycles classes we’ve discussed so far. Using them incorrectly can cause severe concurrency problems.
>
> ------
>
> **NOTE** **Object lifecycle and Dependency Injection Frameworks**
>
> Dependency Injection frameworks can create thread safe, transactional SqlSessions and mappers and inject them directly into your beans so you can just forget about their lifecycle. You may want to have a look at MyBatis-Spring or MyBatis-Guice sub-projects to know more about using MyBatis with DI frameworks.
>
> ------
>
> #### SqlSessionFactoryBuilder
>
> This class can be instantiated, used and thrown away. There is no need to keep it around once you’ve created your SqlSessionFactory. Therefore the best scope for instances of SqlSessionFactoryBuilder is method scope (i.e. a local method variable). You can reuse the SqlSessionFactoryBuilder to build multiple SqlSessionFactory instances, but it’s still best not to keep it around to ensure that all of the XML parsing resources are freed up for more important things.
>
> #### SqlSessionFactory
>
> Once created, the SqlSessionFactory should exist for the duration of your application execution. There should be little or no reason to ever dispose of it or recreate it. It’s a best practice to not rebuild the SqlSessionFactory multiple times in an application run. Doing so should be considered a “bad smell”. Therefore the best scope of SqlSessionFactory is application scope. This can be achieved a number of ways. The simplest is to use a Singleton pattern or Static Singleton pattern.
>
> #### SqlSession
>
> Each thread should have its own instance of SqlSession. Instances of SqlSession are not to be shared and are not thread safe. Therefore the best scope is request or method scope. Never keep references to a SqlSession instance in a static field or even an instance field of a class. Never keep references to a SqlSession in any sort of managed scope, such as HttpSession of the Servlet framework. If you’re using a web framework of any sort, consider the SqlSession to follow a similar scope to that of an HTTP request. In other words, upon receiving an HTTP request, you can open a SqlSession, then upon returning the response, you can close it. Closing the session is very important. You should always ensure that it’s closed within a finally block. The following is the standard pattern for ensuring that SqlSessions are closed:
>
> ```java
> try (SqlSession session = sqlSessionFactory.openSession()) {  // do work}
> ```
>
> Using this pattern consistently throughout your code will ensure that all database resources are properly closed.
>
> #### Mapper Instances
>
> Mappers are interfaces that you create to bind to your mapped statements. Instances of the mapper interfaces are acquired from the SqlSession. As such, technically the broadest scope of any mapper instance is the same as the SqlSession from which they were requested. However, the best scope for mapper instances is method scope. That is, they should be requested within the method that they are used, and then be discarded. They do not need to be closed explicitly. While it’s not a problem to keep them around throughout a request, similar to the SqlSession, you might find that managing too many resources at this level will quickly get out of hand. Keep it simple, keep Mappers in the method scope. The following example demonstrates this practice.
>
> ```java
> try (SqlSession session = sqlSessionFactory.openSession()) {  BlogMapper mapper = session.getMapper(BlogMapper.class);  // do work}
> ```

### 域和生命周期

理解那些我么将要讨论的多种多样的域和生命周期级别是非常重要的，不正确地使用他们会导致多种多样的并发问题

------

*NOTE* **Object lifecycle and Dependency Injection Frameworks**

DI框架可以创建线程安全，事务型的SqlSessions 和 mappers 然后直接将它们注入到beans中，所以你可以不用考虑他们的生命周期．你可能需要看一下MyBatis-Spring和MyBatis-Guice这两个子项目去了解如何配合DI框架使用MyBatis

------

#### SqlSessionFactoryBuilder

这个类可以被实例化，使用和抛出，在你创造了你的SqlSessionFactory后它就没必要继续被保持了，因此SqlSessionFactoryBuilder实例的最佳的域是方法域（比如作为一个本地方法的变量），你依然可以保留SqlSessionFactoryBuilder用以创造多个的SqlSessionFactory实例，但是为了所有和XML语法分析的资源得以释放去做更重要的事，我们最好还是不要保留他．

#### SqlSessionFactory

一旦被创建，SqlSessionFactory就应该在你整个应用执行期间存在，我们甚至几乎都没有理由去销毁或者重新创造它，在一个应用运行期间不重新构建SqlSessionFactory多次是一个最佳的惯例，而不这样做通常会被人们认为是个坏事情．因此最好的SqlSessionFactory域应该是application域，有很多种方式可以做到这样，最简单的就是用一个单例模式或者静态单例模式。

#### SqlSession

每个线程应该拥有它自己的SqlSession实例，SqlSession的实例是不被线程共享的，也是线程不安全的，因此对它来说最好的域是方法域和请求域，永远不要将一个SqlSession实例的引用放在一个静态字段甚至一个类的实例字段中，也永远不要放在任何形式的被管理的域比如Servlet framework的HttpSession中，如果你用了任何形式的web框架，考虑把SqlSession放在一个和HTTP 请求差不多的域中，用另一句话来说就是当收到一个HTTP请求，你就可以开一个SqlSession，然后当返回这个相应的时候，你可以关闭它，关闭这个回话是非常重要的，你必须永远确保会话被关闭在一个finally块，下面是确保会话被关闭的标准模式写法：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {  // do work}
```

#### Mapper Instances

While it’s not a problem to keep them around throughout a request, similar to the SqlSession, you might find that managing too many resources at this level will quickly get out of hand. Keep it simple, keep Mappers in the method scope. The following example demonstrates this practice.

Mappers是那些你创建来用来绑定你的数据库映射语句的接口，Mapper接口的实例是从SqlSession中获得的。如此来说，从技术上讲Mappers域的最大边界和SqlSession的域（requested域）是一样的。然而，对于mapper的实例来说最好的域是方法域，也就是说它必须在使用它的方法中被请求然后用完后被丢弃，他们不需要被明确地关闭。当然和SqlSession一样保留它们在一整个请求也是没问题的，但是你可能会发现在这种层面上管理太多资源将会马上失控，下面的例子证明了这个实践：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {  BlogMapper mapper = session.getMapper(BlogMapper.class);  // do work}
```