---
title: Spring Cloud Zuul网关
photo:
  - >-
    https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg
tags:
  - Spring Cloud
  - Spring Cloud Zuul
  - 微服务网关
categories:
  - - Spring Cloud
    - Zuul
description: 微服务的门卫大爷Zuul
abbrlink: 43513
date: 2018-12-05 22:31:06
---

<center><i>微服务的门卫大爷Zuul</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg)

<!-- more -->

<center><font size="6px">Spring Cloud Zuul网关</font></center>

## Zuul 简单使用

[Zuul官网](https://github.com/Netflix/zuul)

### Zuul 科普

> `Zuul` 是 Netflix 开源的微服务网关，它可以和 `Eureka`，`Ribbon`，`Hystrix`等组件配合使用。    
>
> `Zuul`的核心是一系列的过滤器，这些过滤器可以完成以下功能：
>
> - 系统鉴权：识别每个资源的严正要求，并拒绝那些权限不够的请求。
>
> - 审查与监控：在边缘位置追踪有意义的数据和统计结果，从而带来精确的生产试图。
>
> - 动态路由：动态地将请求路由到不同的后端集群。
>
> - 压力测试：逐渐增加指向集群的流量，借此了解性能。
>
> - 负载分配：为每一种负载类型分配对应的容量，并弃用超出限定值的请求。
>
> - 静态响应处理：在边缘位置直接建立部分响应，从而避免其转发到内部集群。
>
> - 多区域弹性：跨域 AWS Region 进行请求路由，旨在实现 ELB (Elastic Load Balancing) 使用的多样化，以及让系统的边缘更贴近系统的使用者。
>
>
>
>   Spring Cloud 对 Zull 进行了整合与增强。目前 Zull 使用的默认的HTTP客户端是 `Apache Http Client`，也可以使用 `Rest Client` 或者 `okhttp Client`。如果想要使用 `Rest Client`可以设置 `ribbon.restclient.enabled=true`；想要使用 `okhttp Client`可以设置 ` ribbon.okhttp.enabled=true`。



### Zuul 在Spring Cloud的位置

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxw98jh08gj30l40df3zq.jpg)



- 系统的一切请求都将经由`Zuul`网关，这样我们就可以在网关上来做鉴权和路由转发，这样会让整个系统更加的安全和稳定。



### 快速入门

1. 引入依赖

   ```xml
       <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
               </dependency>
   ```


2. application.yml

   ```yaml
   spring:
     application:
       name: gateway
   server:
     port: 9090
   
   ```


3. 启动类加入`@EnableZuulProxy`注解

   ```java
   package com.xushuai;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.cloud.client.SpringCloudApplication;
   import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
   
   @EnableZuulProxy
   @SpringBootApplication
   public class GatewayApplication {
       public static void main(String[] args) {
           SpringApplication.run(GatewayApplication.class);
       }
   }
   
   ```


4. 在`application.yml`中配置路由规则

   ```yaml
   zuul:
     routes:
       # id：这个 id 可以随意命名
       id:
         # 配置路径，这里的路径为服务匹配路径
         path: /provider/**
         # 配置转发到的url，服务提供者的地址
         url : https://localhost:8081
   ```



5. 启动测试

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxw9y2zhyoj30eq03w0st.jpg)

   > 成功访问

6.  注意一个坑，这里我最开始运行时遇到这样一个错误。

   ```java
   The bean 'counterFactory', defined in class path resource [org/springframework/cloud/netflix/zuul/ZuulServerAutoConfiguration$ZuulCounterFactoryConfiguration.class], could not be registered. A bean with that name has already been defined in class path resource [org/springframework/cloud/netflix/zuul/ZuulServerAutoConfiguration$ZuulMetricsConfiguration.class] and overriding is disabled.
   ```

   > 造成这个问题原因是在我的父工程中引入的`Spring Boot`版本为`2.1.0`，然而 `Zuul`还并不支持该版本，所以导致启动保存，只需要降低`Spring Boot`版本就可以了，我将版本降低至2.0.4能够正常运行。



### 服务路由配置

> 在上面的快速入门中，你会发现我们配置的服务地址是采用URL方式配置的，这种方式太局限，无法完成负载均衡等操作，所以我们来换一种配置方式。但在这之前我们需要先引入`Eureka`的客户端依赖

1. 引入`Eureka`依赖

   ```xml
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
   ```

2.  修改`application.yml`配置

   ```yaml
   #
   zuul:
     routes:
       # id：这个 id 可以随意命名
       id:
         # 配置路径，这里的路径为服务匹配路径
         path: /provider/**
         # 配置转发到的url，服务提供者的地址
         url : https://127.0.0.1:8081
   ```

3. 在启动类上加入`@EnableDiscoveryClient`注解

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxwa8f8qdpj30gx050q33.jpg)



4. 启动测试

   > 不出意外，也是能访问成功，而且 `Zuul`默认帮我们是先了负载均衡


### 配置简化和其他配置

1. 路由配置简化

   > 在上面的配置中，这个`id`属性是我们自定义的，但是你会发现这个东西其实毫无作用，是一个可有可无的配置，我们一起来看下简化的配置方式

   ```yaml
   zuul:
     routes:
       # 直接配置 serviceId: path
       provider: /provider/**
   ```



2. `Zuul`默认配置的路由规则就是简化后的配置路由，如果不信，把配置删除试试看，你会大吃一惊，是不是感觉前面说的都是些没用的？

3. 有时候我们某些微服务不希望被外界调用，这时我们可以使用下面的配置忽略掉指定的微服务。

   ```yaml
   zuul:
     # 指定不配置路由的微服务，参数为服务ID集合
     ignored-services: 
       - provider
   ```


4. 路由前缀
   - 可以通过`zuul.prefix`来设置全局的路由前缀。
   - 也可以通过`zuul.strip-prefix`来去除路由前缀，`strip-prefix`也可以添加在某个服务路由的配置中用于去除路由前缀。

## Zuul 过滤器

> Zuul作为网关的其中一个重要功能，就是实现请求的鉴权。而这个动作我们往往是通过Zuul提供的过滤器来实现的。

### ZuulFilter

> ZuulFilter是过滤器的顶级父类。其中定义了四个抽象方法

```java
public abstract ZuulFilter implements IZuulFilter{

    abstract public String filterType();

    abstract public int filterOrder();
    
    boolean shouldFilter();// 来自IZuulFilter

    Object run() throws ZuulException;// IZuulFilter
}
```

1. `filterType`：返回过滤器类型，常用的有以下四种
   - `pre`：前置过滤器
   - `routing`：路由过滤器
   - `error`：异常处理过滤器
   - `post`：后置过滤器，会在`routing`和`error`之后调用，作为返回前的过滤器

2. `filterOrder`：用于定义当前过滤器的执行顺序，值越小优先级越高（可以为负数）。

3. `shouldFilter`：是否执行当前过滤器，返回`true`执行，否则不执行。

4. `run`：过滤器的具体逻辑定义在这个方法内部，通过`RequestContext`定义当前请求是否拦截。


### 过滤器的声明周期

![](https://images.xushuai.fun/18-12-6/64872783.jpg)

1. 请求正常返回

   > 请求到达首先会经过pre类型过滤器，而后到达routing类型，进行路由，请求就到达真正的服务提供者，执行请求，返回结果后，会到达post过滤器。而后返回响应。

2. 请求发生异常
   - 整个过程中，pre或者routing过滤器出现异常，都会直接进入error过滤器，再error处理完毕后，会将请求交给POST过滤器，最后返回给用户。
   - 如果是error过滤器自己出现异常，最终也会进入POST过滤器，而后返回。
   - 如果是POST过滤器出现异常，会跳转到error过滤器，但是与pre和routing不同的时，请求不会再到达POST过滤器了。

3. Zuul 内置过滤器列表

   ![](https://images.xushuai.fun/18-12-6/950130.jpg)


   内置Filter相关参数

   | Filter_Type | Filter_Order |      Filter_Class       |        Filter_Desc         |
   | :---------: | :----------: | :---------------------: | :------------------------: |
   |     pre     |      -3      | ServletDetectionFilter  |   标记处理Servlet的类型    |
   |     pre     |      -2      | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
   |     pre     |      -1      |  FormBodyWrapperFilter  |         包装请求体         |
   |    route    |      1       |       DebugFilter       |        标记调试标志        |
   |    route    |      5       |   PreDecorationFilter   |  处理请求上下文供后续使用  |
   |    route    |      10      |   RibbonRoutingFilter   |     serviceId请求转发      |
   |    route    |     100      | SimpleHostRoutingFilter |        url请求转发         |
   |    route    |     500      |    SendForwardFilter    |      forward请求转发       |
   |    post     |      0       |     SendErrorFilter     |    处理有错误的请求响应    |
   |    post     |     1000     |   SendResponseFilter    |     处理正常的请求响应     |



### 自定义过滤器

1. 只需要编写一个类继承`ZuulFilter`并实现四个方法即可。

   ```java
   package com.xushuai.filter;
   
   import com.netflix.zuul.ZuulFilter;
   import com.netflix.zuul.context.RequestContext;
   import com.netflix.zuul.exception.ZuulException;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
   import org.springframework.http.HttpStatus;
   import org.springframework.stereotype.Component;
   
   /**
    * 自定义过滤器
    */
   @Component
   @Slf4j
   public class MyCustomerFilter extends ZuulFilter {
       @Override
       public String filterType() {
           // 定义当前过滤器类型为：前置过滤器
           return FilterConstants.PRE_TYPE;
       }
   
       @Override
       public int filterOrder() {
           // 定义当前过滤器执行顺序
           return 0;
       }
   
       @Override
       public boolean shouldFilter() {
           // 定义当前过滤器生效
           return true;
       }
   
       /**
        * 过滤器逻辑定义在这个方法内
        */
       @Override
       public Object run() throws ZuulException {
           log.info("My CustomerFilter run.....");
   
           // 获取请求上下文
           RequestContext currentContext = RequestContext.getCurrentContext();
           // true为放行，false为拦截
   //        currentContext.setSendZuulResponse(true);
           currentContext.setSendZuulResponse(false);
           // 设置响应码
           currentContext.setResponseStatusCode(HttpStatus.BAD_REQUEST.value());
   
           // 返回值任意
           return null;
       }
   }
   
   ```


2. 运行测试

   ![](https://images.xushuai.fun/18-12-6/12448675.jpg)



### Zuul路由熔断

1. 通过实现`FallbackProvider`接口来完成路由熔断

   ```java
   package com.xushuai.fallback;
   
   import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
   import org.springframework.http.HttpHeaders;
   import org.springframework.http.HttpStatus;
   import org.springframework.http.MediaType;
   import org.springframework.http.client.ClientHttpResponse;
   import org.springframework.stereotype.Component;
   
   import java.io.ByteArrayInputStream;
   import java.io.IOException;
   import java.io.InputStream;
   
   /**
    * 自定义服务路由熔断
    */
   @Component
   public class ProviderServiceFallback implements FallbackProvider {
   
       /**
        * 这个方法用于需要熔断的服务
        * @return serviceId 服务ID
        */
       @Override
       public String getRoute() {
           return "provider";
       }
   
       /**
        * 这个方法用于指定快速返回的结果
        * @param route 执行熔断的路由
        * @param cause 异常信息
        * @return 自定义返回结果
        */
       @Override
       public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
           return new ClientHttpResponse() {
               @Override
               public HttpHeaders getHeaders() {
                   HttpHeaders headers = new HttpHeaders();
                   headers.setContentType(MediaType.APPLICATION_JSON);
                   return headers;
               }
   
               @Override
               public InputStream getBody() throws IOException {
                   return new ByteArrayInputStream(cause.getMessage().getBytes());
               }
   
               @Override
               public HttpStatus getStatusCode() throws IOException {
                   return HttpStatus.BAD_REQUEST;
               }
   
               @Override
               public int getRawStatusCode() throws IOException {
                   return HttpStatus.BAD_REQUEST.value();
               }
   
               @Override
               public String getStatusText() throws IOException {
                   return cause.getMessage();
               }
   
               @Override
               public void close() {}
           };
       }
   }
   
   ```


2. Zuul的高可用

   > 因为 `Zuul`本身也是一个微服务，只需要启动多个并注册到`Eureka`上即可完成网关的高可用。


