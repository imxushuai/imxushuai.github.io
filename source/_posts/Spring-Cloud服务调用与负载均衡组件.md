---
title: Spring Cloud服务调用与负载均衡组件
photo:
  - >-
    https://www.imxushuai.com/img/asset/006ifTg0gy1fxspi88u31j32a40nygmk.jpg
tags:
  - Spring Cloud
  - Spring Cloud Ribbon
  - Spring Cloud feign
  - 微服务调用
categories:
  - - Spring Cloud
    - Ribbon
  - - Spring Cloud
    - Feign
description: 使用Spring Cloud服务调用实现微服务的调用和负载均衡
abbrlink: 63370
date: 2018-12-03 19:55:10
---

<center><i>使用Spring Cloud服务调用实现微服务的调用和负载均衡</i></center>

![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxspi88u31j32a40nygmk.jpg)

<!-- more -->

<center><font size="6px">Spring Cloud服务调用与负载均衡组件</font></center>


---

## 搭建demo项目

> 项目搭建，只提供相关配置和代码，不深入介绍

### 父工程依赖管理
- pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xushuai</groupId>
    <artifactId>springclouddemo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>provider</module>
        <module>EurekaServer</module>
        <module>consumer</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- SpringCloud依赖管理 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### EurekaServer

1. pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springclouddemo</artifactId>
        <groupId>com.xushuai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>EurekaServer</artifactId>
    <dependencies>
        <!-- Eureka服务端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

</project>
```

2. application.yml'

```yml
server:
  port: 8761 # 端口
spring:
  application:
    # 应用名称，会在Eureka中显示
    name: eureka-server
eureka:
  client:
    # 是否注册自己的信息到EurekaServer，默认是true
    register-with-eureka: false
    # 是否拉取其它服务的信息，默认是true
    fetch-registry: false
    # EurekaServer的地址
    service-url:
      defaultZone: http://localhost:8761/eureka/

```

3. 启动类

```java
package com.xushuai;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class);
    }
}

```

### provider服务提供者

1. pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springclouddemo</artifactId>
        <groupId>com.xushuai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>provider</artifactId>

    <dependencies>
        <!-- Eureka客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

2. application.yml
```yml
server:
  port: 8081
spring:
  application:
    # 应用名称
    name: provider
eureka:
  client:
    service-url: # EurekaServer地址
      defaultZone: http://localhost:8761/eureka
```

3. 启动类
```java
package com.xushuai;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class);
    }
}

```

4. 提供的服务API
```java
package com.xushuai.provider;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProviderController {

    @RequestMapping("/hello")
    public String hello() {
        return "provider -- one";
    }
}
```

### consumer服务消费者

1. pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springclouddemo</artifactId>
        <groupId>com.xushuai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>consumer</artifactId>
    <!-- Eureka客户端 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

2. application.yml
```yml
server:
  port: 8080
spring:
  application:
    # 应用名称
    name: consumer
eureka:
  client:
    service-url: # EurekaServer地址
      defaultZone: http://localhost:8761/eureka
```

3. 启动类
```java
package com.xushuai.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class);
    }
}

```


## Spring Cloud服务调用方式

### 使用RestTemplate

1. 直接使用RestTemplate
  - 配置`RestTemplate`的Bean对象
  ```java
    // 为了方便，这个bean可以配置在启动类中
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```
  
  - 编写controller类
  ```java
  package com.xushuai.consumer;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/consumer")
    public String consumer() {
        return restTemplate.getForObject("http://localhost:8081/hello", String.class);
    }
}
```
  - 调用`consumer`的方法

  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxttnvwbu5j30gp04u0sv.jpg)
  
  > 成功调用服务提供方法，当这种方式看上去有点太。。。但是这种方式我们是通过写死API地址来调用，但实际情况下，服务器的IP地址并不固定，而且一旦并发太高我们会做分布式部署，这时IP地址就不止一个了。所以这种方式，并不能满足需求。
  
2. 通过Eureka客户端获取服务实例列表并调用
  - 在`CounsumerController`中注入`DiscoveryClient`
  > 注意：导包`org.springframework.cloud.client.discovery.DiscoveryClient`
  ```java
    @Autowired
    private DiscoveryClient client;
```
  
  - 修改调用服务代码
  ```java
    @RequestMapping("/consumer")
    public String consumer() {
        // 提供服务ID，获取服务实例列表
        List<ServiceInstance> instances = client.getInstances("provider");
        // 从实例列表中选择一个实例进行调用
        ServiceInstance instance = instances.get(0);
        // 从服务实例中获取ip地址和端口号，并调用服务
        return restTemplate.getForObject("http://" + instance.getHost() +":"+ instance.getPort() +"/hello", String.class);
    }
```
    
  - 访问该方法
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtu1xreo8j30ba03x0sr.jpg)

  > 一样可以访问到，但是这种方式还是有问题，虽然ip地址变成动态获取的，但如果要完成负载均衡，我们需要自己手动实现负载均衡算法，而且调用起来也毕竟麻烦。
  
  3. 使用`Ribbon`完成服务调用和负载均衡
  - 在`RestTemplate`的bean方法上加入`@LoadBalanced`注解
  ```java
     @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```
  
  - 再次修改`ConsumerController`中的`consumer`方法
  ```java
    @RequestMapping("/consumer")
    public String consumer() {
        // url地址，使用服务ID取代ip地址和端口号
        String url = "http://provider/hello";
        // 从服务实例中获取ip地址和端口号，并调用服务
        return restTemplate.getForObject(url, String.class);
    }
```

  - 在运行测试之前，为了能够看到负载均衡的效果，我们再启动一个`provider`，再启动之前修改`ProviderController`中的返回值为`provider -- two`，再将端口号修改为`8082`，或者在`VM Options`中配置。
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtumr3zrsj30fx0433yp.jpg)

  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuna877jj30i704w0t1.jpg)
  
  - 启动测试，访问`ConsumerController`
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuobgspgj30c803cdfv.jpg)

  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuok2rzhj30ar03awei.jpg)
  
  > 多测试会发现，每次访问的返回值都不同，这是因为`Ribbon`默认采用的负载均衡算法为轮询模式。
  
  4. Ribbon实现原理
  - `Ribbon`的实现主要是采用的拦截器机制，`Ribbon`使用`LoadBalancerInterceptor`拦截了`RestTemplate`，在发送请求之前，替换成了具体的IP和端口号
  
  - 我们使用断点调试来稍微看一看
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuvekuqfj30z30crmzj.jpg)

  > 请求被拦截处理，继续进入`execute`方法
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuz4dpssj30w807u0u5.jpg)
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtuz4dpssj30w807u0u5.jpg)
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtv2wrnu4j30pp045jrn.jpg)
  
  > 采用默认的负载均衡器，进入`chooseServer`方法
  
  ![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxtv6fxqzgj30st077q4a.jpg)
  
  > 看到这里就已经差不多了解`Ribbon`负载均衡调用的原理了
  
  5. 设置轮询策略
  ```yml
provider:
  ribbon:
    # 设置轮询策略为随机
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule 
```
  

### 使用Fegin完成服务调用和负载均衡

1. 在`consumer`中引入`Fegin`的依赖
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
```

2. 在`consumer`的启动类上`@EnableFeignClients`注解
```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

3. 编写服务提供者接口
```java
package com.xushuai.consumer;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
// name为服务ID
@FeignClient(name = "provider")
public interface FeignProviderService {

    @RequestMapping("hello")
    public String hello();
}
```

4. 将`FeignProviderService`注入到`ConsumerController`中
```java
    @Autowired
    private FeignProviderService service;
```

5. 调用远程服务
```java
    @RequestMapping("/consumer")
    public String consumer() {
        // 直接调用即可
        return service.hello();
    }
```

6. 访问
> 会发现其默认负载均衡策略为轮询，而且其底层使用的负载均衡策略也是`Ribbon`的轮询规则类

## 重试机制

Eureka的服务治理强调了CAP原则中的AP，即可用性和可靠性。它与Zookeeper这一类强调CP（一致性，可靠性）的服务治理框架最大的区别在于：Eureka为了实现更高的服务可用性，牺牲了一定的一致性，极端情况下它宁愿接收故障实例也不愿丢掉健康实例，正如我们上面所说的自我保护机制。

但是，此时如果我们调用了这些不正常的服务，调用就会失败，从而导致其它服务不能正常工作！这显然不是我们愿意看到的。

### 开启重试

1. 引入依赖
```xml
    <dependency>
        <groupId>org.springframework.retry</groupId>
        <artifactId>spring-retry</artifactId>
    </dependency>
```

2. application.yml
```yml
spring:
  cloud:
    loadbalancer:
      retry:
        # 开启Spring Cloud的Retry
        enabled: true 
provider:
  ribbon:
    # Ribbon的连接超时时间
    ConnectTimeout: 250 
    # Ribbon的数据读取超时时间
    ReadTimeout: 1000 
    # 是否对所有操作都进行重试
    OkToRetryOnAllOperations: true 
    # 切换实例的重试次数
    MaxAutoRetriesNextServer: 1 
    # 对当前实例的重试次数
    MaxAutoRetries: 1 
```
