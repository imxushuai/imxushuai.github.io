---
title: Spring Boot自动配置原理
tags: 
  - Spring Boot
  - Spring Boot原理
categories: Spring Boot
description: 为什么Spring Boot配置那么少？让我们一起走进Spring Boot自动配置
photos:
  - >-
    https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxq697a7goj30sg08fdgl.jpg
abbrlink: 60852
date: 2018-11-28 22:32:04
---

<center><i>为什么Spring Boot配置那么少？让我们一起走进Spring Boot自动配置</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxq697a7goj30sg08fdgl.jpg)

<!-- more -->

<center><font size="6px">Spring Boot自动配置原理</font></center>


---
### Spring Boot启动类
```java
package com.xushuai.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Spring Boot标准启动类
 */
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

Spring Boot启动类的关键:
- `@SpringBootApplication`：Spring Boot应用注解
- `Main函数`：Spring Boot程序入口


### @SpringBootApplication

> 查看`@SpringBootApplication`的源码，可以发现在该注解上还有三个关键的注解   
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ......
}
```

1. `@SpringBootConfiguration`：其实该类的作用和`@Configuration`的作用是一样的，只是为了让语义更加明确，声明这是一个Spring Boot应用的配置类，且该注解一个项目中只能有一个。
2. `@EnableAutoConfiguration`：开启自动化配置，这是为什么Spring Boot能帮我们自动配置的关键所在，该注解会配合`spring-boot-autoconfigure`完成自动化配置过程。
> 官方给出的描述：   第二级的注解@EnableAutoConfiguration，告诉SpringBoot基于你所添加的依赖，去“猜测”你想要如何配置Spring。比如我们引入了spring-boot-starter-web，而这个启动器中帮我们添加了tomcat、SpringMVC的依赖。此时自动配置就知道你是要开发一个web应用，所以就帮你完成了web及SpringMVC的默认配置了！

3. `@ComponentScan`：Spring的包扫描，该注解如果不声明`basePackages`会以声明该注解的类的路径为基础包扫描路径。


### Spring Boot Main函数入口
> 查看源码，在执行run方法的过程中，会对Spring容器进行初始化，这个初始化会加载当前项目所有的bean对象，包括引入的jar包中的bean对象（实质是加载`spring.factories`文件中声明的所有类到spring容器）。   
![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxo5vv9r4bj30gq05xwet.jpg)


1. Spring Boot会加载依赖中的`spring-boot-autoconfigure`，该依赖中进行了大量的默认配置，包括很多主流的开源框架（这里不一一举例，可以点开源码进行查看）
![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxo4k4veg9j30h905nwex.jpg)

2. `spring-boot-autoconfigure`是怎么完成自动配置的？在该项目中，我们会发现有很多`AutoConfiguration` 类，所有的默认配置都是在这些类中完成的，在这些类上你会发现都有一些以`Conditional`开头的注解，这些注解就是用来控制当前类中的配置是否生效的注解 （以`RedisRepositoriesAutoConfiguration `为例）。
```java
@Configuration
@ConditionalOnClass(EnableRedisRepositories.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnProperty(prefix = "spring.data.redis.repositories", name = "enabled", havingValue = "true", matchIfMissing = true)
@ConditionalOnMissingBean(RedisRepositoryFactoryBean.class)
@Import(RedisRepositoriesAutoConfigureRegistrar.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class RedisRepositoriesAutoConfiguration {

}
```
3. `Conditional`相关注解   
   3.1 `ConditionalOnClass`：此项目下存在参数列表中的类时，则自动配置生效。
   3.2 `ConditionalOnBean`：此项目下存在参数列表中的Bean时，则自动配置生效。
   3.3 `ConditionalOnProperty`：此项目下存在参数列表中的配置文件时，则自动配置生效（该注解还可以进一步定制，具体可以自行查看该配置的源码）。
   3.4 `ConditionalOnMissingBean`：此项目中没有参数列表中的Bean，则自动配置生效（防止重复配置）。

4. 默认配置覆盖，主要有两种方法   
   4.1 自定义相同类型的`Bean`覆盖掉默认配置中的`Bean`。   
   4.2 在application.yaml中覆盖掉指定`Bean`的属性(可以在`spring-boot-autoconfigure`中找到想要覆盖的属性的key，然后在自己的项目重新声明该key的值就可以了)。


> Spring Boot自动配置原理就大致如此，更多细节请阅读Spring Boot源码和官方文档。   
[Spring Boot官网](https://spring.io/projects/spring-boot)
