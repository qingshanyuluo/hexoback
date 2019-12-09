---
title: springCloud入门
abbrlink: 13124536
date: 2019-12-1 10:43:39
tag: 
    - springCloud
    - java
    - 分布式
category: java
---

> Spring Cloud为开发人员提供了工具来快速构建分布式系统中一些常见的模式(例如配置管理、服务发现、断路器、智能路由、micro-proxy、 control bus控制总线、one-time tokens 、global locks 全局锁、 leadership election ,分布式会话、集群状态)。组织分布式系统使之模板化(Coordination of distributed systems leads to boiler plate patterns )，使用Spring Cloud开发者可以快速建立实现这些模式的服务和应用程序。他们将适用于任何分布式环境中,包括开发人员自己的笔记本电脑,物理机( bare metal data centers)数据中心,云计算等和管理平台。 

## Features

Spring Cloud focuses on providing good out of box experience for typical use cases and extensibility mechanism to cover others.

- Distributed/versioned configuration  【分布式以及版本化的配置】
- Service registration and discovery【服务注册与发现】
- Routing【路由】
- Service-to-service calls【服务调用】
- Load balancing【负载均衡】
- Circuit Breakers【断路器】
- Global locks
- Leadership election and cluster state
- Distributed messaging【分布式消息】

## 服务注册与发现

统一化的管理，可以实现各个微服务间良好的解耦，服务提供者和消费者都注册到Eureka服务器上，浏览器访问消费者地址，消费者会通过Eureka服务器上服务提供者的名字，找到提供者的真实路径，进行调用。高可用时可以搭建几个Eureka服务器，互相注册，复制各自信息。 

贴一下关键代码：

## eureka server：

项目开始前更改host 将 discovery 映射到127.0.0.1

m1\src\main\java\xyz\lennon\EurekaApplication.java

```java
package xyz.lennon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {
  public static void main(String[] args) {
    SpringApplication.run(EurekaApplication.class, args);
  }
}
```

application.yml

```yml
server:
  port: 8761                    # 指定该Eureka实例的端口

eureka:
  instance:
    hostname: discovery         # 指定该Eureka实例的主机名
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 用户模块

m2\src\main\java\xyz\lennon\UserApplication.java

```java
package xyz.lennon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class UserApplication {
  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
```

application.yml

```yml
server:
  port: 8001
spring:
  application:
    name: microservice-provider-user    # 项目名称尽量用小写
  jpa:
    generate-ddl: false
    show-sql: true
    hibernate:
      ddl-auto: none
  datasource:                           # 指定数据源
    platform: h2                        # 指定数据源类型
    schema: classpath:schema.sql        # 指定h2数据库的建表脚本
    data: classpath:data.sql            # 指定h2数据库的insert脚本
logging:
  level:
    root: INFO
    org.hibernate: INFO
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
    org.hibernate.type.descriptor.sql.BasicExtractor: TRACE
    com.itmuch.youran.persistence: ERROR
eureka:
  client:
    serviceUrl:
      defaultZone: http://discovery:8761/eureka/    # 指定注册中心的地址
  instance:
    preferIpAddress: true
```

启动上述两个模块，打开 http://discovery:8761/ 就可以看到监控页面注册的用户模块，用户模块也可以同时启动在不同端口上，注册到eureka，值得注意的是，应用名字如果相同，注册中心就会认为是同一个服务的不同实例。

## 负载均衡

借助**Ribbon**实现负载均衡， 规则用默认的。

开启eureka server 和用户模块，为了模拟负载均衡，在不同的端口启动多个用户模块

### 调用用户模块的模块

m3\src\main\java\xyz\lennon\MovieRibbonApplication.java


```java
package xyz.lennon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public class MovieRibbonApplication {
  /**
   * 实例化RestTemplate，通过@LoadBalanced注解开启均衡负载能力.
   * @return restTemplate
   */
  @Bean
  @LoadBalanced
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }

  public static void main(String[] args) {
    SpringApplication.run(MovieRibbonApplication.class, args);
  }
}
```

m3\src\main\java\xyz\lennon\RibbonService.java

```java
package xyz.lennon;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

/**
 * dfd
 */
@Service
public class RibbonService {
  @Autowired
  private RestTemplate restTemplate;

  public User findById(Long id) {
    return this.restTemplate.getForObject("http://microservice-provider-user/" + id, User.class);
  }
}
```

可以看到restTemplate是基于注册的应用名调用的

```yml
server:
  port: 8010
spring:
  application:
    name: microservice-consumer-movie-ribbon
eureka:
  client:
    serviceUrl:
      defaultZone: http://discovery:8761/eureka/
  instance:
    preferIpAddress: true
```

多次请求 http://localhost:8010/ribbon/1  观察后台日志发现负载均衡起作用了

## 熔断器

**Hystrix** 防止在服务之间调用失败时“卡死”的情况，停止无意义的调用并给出监控信息

对比上文模块的改进：

### 调用用户模块的模块

m4\src\main\java\xyz\lennon\MovieRibbonHystrixApplication.java


```java
package xyz.lennon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 * 使用@EnableCircuitBreaker注解开启断路器功能
 * 
 * @author eacdy
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class MovieRibbonHystrixApplication {
  /**
   * 实例化RestTemplate，通过@LoadBalanced注解开启均衡负载能力.
   * @return restTemplate
   */
  @Bean
  @LoadBalanced
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }

  public static void main(String[] args) {
    SpringApplication.run(MovieRibbonHystrixApplication.class, args);
  }
}
```

m3\src\main\java\xyz\lennon\RibbonHystrixService.java

```java
package xyz.lennon;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class RibbonHystrixService {
  @Autowired
  private RestTemplate restTemplate;
  private static final Logger LOGGER = LoggerFactory.getLogger(RibbonHystrixService.class);

  /**
   * 使用@HystrixCommand注解指定当该方法发生异常时调用的方法
   * @param id id
   * @return 通过id查询到的用户
   */
  @HystrixCommand(fallbackMethod = "fallback")
  public User findById(Long id) {
    return this.restTemplate.getForObject("http://microservice-provider-user/" + id, User.class);
  }

  /**
   * hystrix fallback方法
   * @param id id
   * @return 默认的用户
   */
  public User fallback(Long id) {
    RibbonHystrixService.LOGGER.info("异常发生，进入fallback方法，接收的参数：id = {}", id);
    User user = new User();
    user.setId(-1L);
    user.setUsername("default username");
    user.setAge(0);
    return user;
  }
}
```

可以看到restTemplate是基于注册的应用名调用的

```yml
server:
  port: 8011
spring:
  application:
    name: microservice-consumer-movie-ribbon-with-hystrix
eureka:
  client:
    serviceUrl:
      defaultZone: http://discovery:8761/eureka/
  instance:
    hostname: ribbon          # 此处，preferIpAddress不设置或者设为false，不能设为true，否则影响turbine的测试。turbine存在的问题：eureka.instance.hostname一致时只能检测到一个节点，会造成turbine数据不完整
```

关闭用户模块，并没有无响应，而是 获得结果：`{"id":-1,"username":"default username","age":0}`，另外日志打印：`c.i.c.s.u.service.RibbonHystrixService : 异常发生，进入fallback方法，接收的参数：id = 1` 。 

参考教程： [使用Spring Cloud与Docker实战微服务](http://book.itmuch.com/)

