---
title: Spring Cloud Eureka
photo:
  - >-
    https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg
tags:
  - Spring Cloud
  - Eureka
categories:
  - Spring Cloud
  - Eureka
description: Spring Cloud服务注册与发现组件：Eureka
abbrlink: 13311
date: 2018-12-02 21:27:17
---

<center><i>Spring Cloud服务注册与发现组件：Eureka</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg)

<!-- more -->

<center><font size="6px">Spring Cloud Eureka</font></center>


---

## Spring Cloud Eureka 简介

### 什么是Eureka？

Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和Service Discovery实现。也是springcloud体系中最重要最核心的组件之一。

### Eureka核心组件

- Eureka Server 注册中心

- Eureka Client 服务注册

> Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现。Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server，并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。Spring Cloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。   
Eureka由两个组件组成：Eureka服务器和Eureka客户端。Eureka服务器用作服务注册服务器。Eureka客户端是一个java客户端，用来简化与服务器的交互、作为轮询负载均衡器，并提供服务的故障切换支持。Netflix在其生产环境中使用的是另外的客户端，它提供基于流量、资源利用率以及出错状态的加权负载均衡。

### Eureka角色构成

Eureka的基本架构，由3个角色组成：

1. Eureka Server
  - 提供服务注册和发现

2. Service Provider
  - 服务提供方
  - 将自身服务注册到Eureka，从而使服务消费方能够找到

3. Service Consumer
  - 服务消费方
  - 从Eureka获取注册服务列表，从而能够消费服务


## JavaDemo

> Spring Cloud版本采用 Finchley.SR1

### 编写Eureka Server

1. pom.xml
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies>
 
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```
> 主要依赖:
>  - eureka-server
> - spring-cloud-start-config

2. application.yml
```yml
eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
    service-url:
      defaultZone : https://localhost:8761/eureka/
server:
  port: 8761
```


> `fetch-registry`：是否从Eureka Server上获取注册信息，默认为true，此处建议修改成 false （单机设置的意义不大，如果设置成 true 启动会去抓取一次注册表，获取不到更新缓存就会出错（该错误不影响 eureka 正常使用））   
> `register-with-eureka`：是否将自己注册到Eureka Server，默认为true。默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为。如果不配置此项，启动时会报Cannot execute request on any known server。是因为在启动时，服务是还没有启动好的，但不影响正常启动。启动好后，等发送一次心跳包后，就正常注册了，只不过是自己注册到自己上，意思就是当前服务即是server又是client。   
> `defaultZone`：设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是https://localhost:8761/eureka ；多个地址可使用 , 分隔。 

**其他配置项：**

```yml
eureka:
  instance:
    prefer-ip-address: true
  # 生产环境中官方是不建议修改默认配置，因为那样会破坏 eureka server 的保护模式
  server:
    # 关闭保护模式（生产环境不建议修改，开发环境最好修改为false，默认为true）
    enable-self-preservation: false
    # 清理间隔（默认是60 * 1000 毫秒）（生产环境不建议修改）
    eviction-interval-timer-in-ms: 60000
    # Eureka 拉取服务列表时间（默认：30秒）（生产环境不建议修改）
    remote-region-registry-fetch-interval: 5
  client:
    # 从 Eureka 服务器端获取注册信息的间隔时间（默认：30秒）
    registry-fetch-interval-seconds: 5

```

3. 在启动类上加入@EnableEurekaServer注解

```java
package com.xushuai.springclouddemo;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
 
@SpringBootApplication
@EnableEurekaServer
public class SpringCloudDemoApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudDemoApplication.class, args);
    }
}
```

4. 访问`https://localhost:8761`,查看Eureka的web ui

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqpjdyvuj31ds0okjtc.jpg)


### 编写Eureka Client

1. pom.xml
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
 
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

> 主要依赖为：eureka-client   
注意：这里记得依赖上 spring-boot-starter-web，这样启动的时候，该项目才会处于运行状态（运行在tomcat容器中），否则你运行后，什么都没干就关掉了。

2. application.yml
```yml
spring:
  application:
    name: eureka-client-1
eureka:
  client:
    service-url:
      defaultZone: https://localhost:8761/eureka
#  instance:
#    hostname: ipOrHostname
server:
  port: 8001

```
> 配置注册中心地   
instance.hostname：这配置的为web ui中客户端的链接地址

3. 在启动类上添加@EnableDiscoveryClient注解
```java
package com.xushuai.eurekaclient;
 
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
 
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClientApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

4. 运行，查看web ui,你会发现在`instances currently registered with Eureka`会多出一个application,名称为`spring.application.name`的值

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqon4cnfj31fr0j376f.jpg)

### Eureka HA(高可用)

1. 使用idea配置两个server
  - Ctrl + shift + A 输入 Edit Configuration
  
  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqr131ikj30h90bzjs5.jpg)

  - 复制启动类
  
  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqr19g83j30dj0iyt9r.jpg)

  - 配置VM options
  
  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqr1dpovj30an06lq33.jpg)

  - 注释掉yml中的server.port并在启动前修改defaultZone为另一个Eureka Server（互相注册。即启动1时修改端口为8761，启动2时修改端口为8762，当然在实际环境当中肯定都是事先配置好的，我这里使用的是同一块代码，所以需要在启动前修改）。
  
  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqr1hqq8j30f708vt96.jpg)

2. 分别启动两个Server 和 Client，并查看web ui

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsquy00gfj314p0kbgnr.jpg)

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqux2b0wj315m0khjtl.jpg)

> 可以看到访问两个server都可以看到服务的消费方

3. 关掉其中任意一个server端

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqwur7ecj30qs0dzdgf.jpg)

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqwxpmpkj31b40j50uy.jpg)

> Eureka-Server已经挂掉，而Eureka-Server（1）依旧正常运行，且能够显示消费方。（红字部分为关闭保护模式后的提示信息，无须在意）

4. 但此时工作其实还没有完全完成，我们重启client会发现，并不能连接上Eureka-Server，因为此时Eureka-Server已经关闭，所以此时我们要在client端配置上两个Eureka-Server的地址，修改client的`application.yml`文件（以逗号分隔）。

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqytrpzxj30ky02wt8q.jpg)

5. 当我们的Server节点超过2个，需要让每两个节点互相注册（和双Server节点操作类似，只需要在defaultZone配置另外两个节点的Eureka地址即可）。

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxsqznipjrj31060a5djf.jpg)

