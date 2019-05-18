---
title: Spring Cloud Hystrix熔断器入门
photo:
  - >-
    https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtylsthm1j30mb08fac1.jpg
tags:
  - Spring Cloud
  - Hystrix
categories:
  - - Spring Cloud
    - Hystrix
description: 微服务的安全措施Hystrix熔断器
abbrlink: 32084
date: 2018-12-03 23:04:48
---

<center><i>微服务的安全措施Hystrix熔断器</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtylsthm1j30mb08fac1.jpg)

<!-- more -->

<center><font size="6px">Spring Cloud Hystrix熔断器入门</font></center>


---
## 熔断器

分布式系统中经常会出现某个基础服务不可用造成整个系统不可用的情况, 这种现象被称为服务雪崩效应. 为了应对服务雪崩, 一种常见的做法是手动服务降级. 而Hystrix的出现,给我们提供了另一种选择.

### 雪崩效应
  > 服务雪崩效应是一种因 服务提供者 的不可用导致 服务调用者 的不可用,并将不可用 逐渐放大 的过程

  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtyw891u2j30ak0cyaep.jpg)
  
  A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。
    
### 什么是熔断器?

  > 它可以实现快速失败，如果它在一段时间内侦测到许多类似的错误，会强迫其以后的多个调用快速失败，不再访问远程服务器，从而防止应用程序不断地尝试执行可能会失败的操作，使得应用程序继续执行而不用等待修正错误，或者浪费CPU时间去等到长时间的超时产生。熔断器也可以使应用程序能够诊断错误是否已经修正，如果已经修正，应用程序会再次尝试调用操作。
  
  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtyzbf2snj30dv06zwgz.jpg)
  
## Spring Cloud熔断器

### Hystrix介绍

> Hystrix [hɪst'rɪks]的中文含义是豪猪, 因其背上长满了刺,而拥有自我保护能力. Netflix的 Hystrix 是一个帮助解决分布式系统交互时超时处理和容错的类库, 它同样拥有保护系统的能力。

1. Hystrix特性
  - 熔断器机制
    > 断路器很好理解, 当Hystrix Command请求后端服务失败数量超过一定比例(默认50%), 断路器会切换到开路状态(Open). 这时所有请求会直接失败而不会发送到后端服务. 断路器保持在开路状态一段时间后(默认5秒), 自动切换到半开路状态(HALF-OPEN). 这时会判断下一次请求的返回情况, 如果请求成功, 断路器切回闭路状态(CLOSED), 否则重新切换到开路状态(OPEN). Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且断路器有自我检测并恢复的能力.

  - Fallback
    > Fallback相当于是降级操作. 对于查询操作, 我们可以实现一个fallback方法, 当请求后端服务出现异常的时候, 可以使用fallback方法返回的值. fallback方法的返回值一般是设置的默认值或者来自缓存.

  - 资源隔离
    > 在Hystrix中, 主要通过线程池来实现资源隔离. 通常在使用的时候我们会根据调用的远程服务划分出多个线程池. 例如调用产品服务的Command放入A线程池, 调用账户服务的Command放入B线程池. 这样做的主要优点是运行环境被隔离开了. 这样就算调用服务的代码存在bug或者由于其他原因导致自己所在线程池被耗尽时, 不会对系统的其他服务造成影响. 但是带来的代价就是维护多个线程池会对系统带来额外的性能开销. 如果是对性能有严格要求而且确信自己调用服务的客户端代码不会出问题的话, 可以使用Hystrix的信号模式(Semaphores)来隔离资源.

### Demo

1. 引入`Hystrix`依赖到`consumer`
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
```

2. 在`consumer`启动类上加入`@EnableCircuitBreaker`
```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
// 可以使用@SpringCloudApplication代替上方三个注解
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

3. 修改`ConsumerController`中的`consumer`方法，加入`@HystrixCommand`注解
```java
    @RequestMapping("/consumer")
    // 配置开启熔断，并设置降级调用方法
    @HystrixCommand(fallbackMethod = "consumerFallback")
    public String consumer() {
        String url = "https://provider/hello";
        return restTemplate.getForObject(url, String.class);
    }
```

4. 编写失败Fallback方法，方法名为配置的`fallbackMethod`的值
```java
    public String consumerFallback() {
        return "服务器忙，请稍后重试！！";
    }
```

5. 为了方便测试，在provider方法中使其提供的服务方法睡眠3秒钟
```java
    @RequestMapping("/hello")
    public String hello() throws InterruptedException {
        Thread.sleep(3000);
        return "provider -- one";
    }
```

6. 运行测试

  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtznnuth1j30as032dfv.jpg)

  > 会发现很快服务端就返回了`consumerFallback`方法的结果，这说明设置的`Hystrix`已经生效，而且很明显有默认的请求超时时间，该时间默认为：`1000`毫秒。

  ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxtztnweehj310607uwgl.jpg)
  
7. 设置`Hystrix`超时时间
```yml
hystrix:
  command:
  	default:
        execution:
          isolation:
            thread:
              # 设置hystrix的超时时间为6000ms
              timeoutInMillisecond: 6000 
```

