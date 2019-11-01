---
title: Spring Cloud简述
photo:
  - >-
    https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg
tags: Spring Cloud
categories: Spring Cloud
description: 让我们一起走进微服务的世界！
abbrlink: 34585
date: 2018-12-02 21:04:41
---

<center><i>让我们一起走进微服务的世界！</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspi88u31j32a40nygmk.jpg)

<!-- more -->

<center><font size="6px">Spring Cloud</font></center>


---

## 为什么要用Spring Cloud

微服务是一种架构方式，最终肯定需要技术架构去实施。

微服务的实现方式很多，但是最火的莫过于Spring Cloud了。为什么？

- 后台硬：作为Spring家族的一员，有整个Spring全家桶靠山，背景十分强大。
- 技术强：Spring作为Java领域的前辈，可以说是功力深厚。有强力的技术- 团队支撑，一般人还真比不了
- 群众基础好：可以说大多数程序员的成长都伴随着Spring框架，试问：现在有几家公司开发不用Spring？SpringCloud与Spring的各个框架无缝整合，对大家来说一切都是熟悉的配方，熟悉的味道。
- 使用方便：相信大家都体会到了SpringBoot给我们开发带来的便利，而SpringCloud完全支持SpringBoot的开发，用很少的配置就能完成微服务框架的搭建

## 简介
SpringCloud是Spring旗下的项目之一，官网地址：http://projects.spring.io/spring-cloud/

Spring最擅长的就是集成，把世界上最好的框架拿过来，集成到自己的项目中。

SpringCloud也是一样，它将现在非常流行的一些技术整合到一起，实现了诸如：配置管理，服务发现，智能路由，负载均衡，熔断器，控制总线，集群状态等等功能。其主要涉及的组件包括：

netflix

- Eureka：注册中心
- Zuul：服务网关
- Ribbon：负载均衡
- Feign：服务调用
- Hystix：熔断器
- ......
- 
## 系统架构演变

### 集中式架构
当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是影响项目开发的关键。

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspqzahzij309o0f2t8v.jpg)

存在的问题：

- 代码耦合，开发维护困难
- 无法针对不同模块进行针对性优化
- 无法水平扩展
- 单点容错率低，并发能力差

### 垂直拆分
当访问量逐渐增大，单一应用无法满足需求，此时为了应对更高的并发和业务需求，我们根据业务功能对系统进行拆分：

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspqyce20j30em0ewgls.jpg)

优点：
- 系统拆分实现了流量分担，解决了并发问题
- 可以针对不同模块进行优化
- 方便水平扩展，负载均衡，容错率提高
缺点：

- 系统间相互独立，会有很多重复开发工作，影响开发效率

### 分布式服务

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式调用是关键。

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspqygahvj30eo0dwt96.jpg)

优点：

- 将基础服务进行了抽取，系统间相互调用，提高了代码复用和开发效率
缺点：

- 系统间耦合度变高，调用关系错综复杂，难以维护

### SOA架构（面向服务架构）

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。（阿里系的dubbo就是经典的SOA架构）

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspqyqi44j30km0bp7b7.jpg)

以前出现了什么问题？

- 服务越来越多，需要管理每个服务的地址
- 调用关系错综复杂，难以理清依赖关系
- 服务过多，服务状态难以管理，无法根据服务情况动态管理

服务治理要做什么？

- 服务注册中心，实现服务自动注册和发现，无需人为记录服务地址
- 服务自动订阅，服务列表自动推送，服务调用透明化，无需关心依赖关系
- 动态监控服务状态监控报告，人为控制服务状态

缺点：

- 服务间会有依赖关系，一旦某个环节出错会影响较大
- 服务关系复杂，运维、测试部署困难，不符合DevOps思想

### 微服务

前面说的SOA，英文翻译过来是面向服务。微服务，似乎也是服务，都是对系统进行拆分。因此两者非常容易混淆，但其实缺有一些差别：

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/006ifTg0gy1fxspqz4o57j30m90c10zx.jpg)

微服务的特点：

- 单一职责：微服务中每一个服务都对应唯一的业务能力，做到单一职责
- 微：微服务的服务拆分粒度很小，例如一个用户管理就可以作为一个服务。每个服务虽小，但“五脏俱全”。
- 面向服务：面向服务是说每个服务都要对外暴露服务接口API。并不关心服务的技术实现，做到与平台和语言无关，也不限定用什么技术实现，只要提供Rest的接口即可。
- 自治：自治是说服务间互相独立，互不干扰
  - 团队独立：每个服务都是一个独立的开发团队，人数不能过多。
  - 技术独立：因为是面向服务，提供Rest接口，使用什么技术没有别人干涉
  - 前后端分离：采用前后端分离开发，提供统一Rest接口，后端不用再为PC、移动段开发不同接口
  - 数据库分离：每个服务都使用自己的数据源
  - 部署独立，服务间虽然有调用，但要做到服务重启不影响其它服务。有利于持续集成和持续交付。每个服务都是独立的组件，可复用，可替换，降低耦合，易维护

