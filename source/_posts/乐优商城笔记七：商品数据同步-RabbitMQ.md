---
title: 乐优商城笔记七：商品数据同步(RabbitMQ)
tags:
  - 乐优商城
categories:
  - 乐优商城
date: 2019-04-02 21:19:23
---

<center><i>使用RabbitMQ完成商品数据同步</i></center>

![](https://images.xushuai.fun/18-12-17/46731258.jpg )

<!-- more -->

> 商品增删改时，同步更新索引库、静态页等。

### 业务分析

#### 发送方 - 商品微服务

- 发送同步数据消息的时机

  当商品发生了增、删、改操作时，发送数据同步消息，通知其他微服务进行数据同步。

- 消息中包含的内容

  考虑到商品数据非常的多，包含了商品SPU、SKU、商品详情等；所以发送时，仅发送`商品ID`，其他微服务拿到ID在进行查询，使用查询得到的数据进行更新操作。

#### 接收方 - 搜索微服务、商品详情页微服务

- 搜索微服务

  当增加和修改时，对索引库进行更新操作；当删除时，删除指定商品的索引。

- 商品详情页微服务

  增加商品时，创建对应商品详情页；修改时，根据更新后的数据创建新的商品详情页，并删除之前的商品详情页；删除时，直接删除指定商品详情页。

### 发送方 - 商品微服务

#### 准备工作

##### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

##### 配置application.yml

```yaml
# 在Spring配置中新增RabbitMQ相关配置
spring:
  rabbitmq:
    # 主机地址
    host: 192.168.136.103
    # 用户名、密码
    username: leyou
    password: leyou
    # 虚拟主机
    virtual-host: /leyou
    template:
      retry:
        enabled: true
        initial-interval: 10000ms
        max-interval: 60000ms
        multiplier: 2
      exchange: leyou.item.exchange
    publisher-confirms: true
```

**配置说明**

- template：有关`AmqpTemplate`的配置
  - retry：失败重试
    - enabled：开启失败重试
    - initial-interval：第一次重试的间隔时长
    - max-interval：最长重试间隔，超过这个间隔将不再重试
    - multiplier：下次重试间隔的倍数，此处是2即下次重试间隔是上次的2倍
  - exchange：缺省的交换机名称，此处配置后，发送消息如果不指定交换机就会使用这个
- publisher-confirms：生产者确认机制，确保消息会正确发送，如果发送失败会有错误回执，从而触发重试

> 注意：
>
>  	1. 该配置是放在 `spring`配置下的。新增配置时注意层级。
>  	2. `rabbitmq`中的`leyou`账号，需要提前创建好。

##### 统一定义RoutingKey

> 我这里把`QueueName`和`ExchangeName`都定义在了Common工程中的常量类`LeyouConstants`中。

- `LeyouConstants`新增常量

  ```java
  /**
   * reabbitmq queue and exchange NAME
   */
  public static final String QUEUE_INSERT_ITEM = "ly.item.insert";
  public static final String QUEUE_UPDATE_ITEM = "ly.item.update";
  public static final String QUEUE_DELETE_ITEM = "ly.item.delete";
  public static final String EXCHANGE_DEFAULT_ITEM = "ly.item.exchange";
  ```



#### GoodsService新增逻辑

> 在`GoodsService`中的增删改中新增发送消息逻辑

##### 消息发送方法

```java
    /**
     * 发送同步数据消息
     *
     * @param id         商品ID
     * @param routingKey rabbitmq routingKey
     */
    private void sendMessage(Long id, String routingKey) {
        // 发送消息
        try {
            amqpTemplate.convertAndSend(routingKey, id);
        } catch (Exception e) {
            log.error("[{}]商品消息发送异常，商品id：{}", routingKey, id, e);
        }
    }
```

> 吞掉异常，防止发送消息失败影响到正常的流程。

##### 在增删改方法中新增逻辑

> 在增删改方法中，调用发送消息的方法

- 新增

  ![](<https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190516235042.png>)

- 修改

  ![](<https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190516235103.png>)

- 删除

  ![](<https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190516235120.png>)



### 接收方

#### 准备工作

> 准备工作的内容，需要分别在搜索微服务和商品详情页微服务各做一次。

##### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

##### 配置application.yml

```java
spring:
  # RabbitMQ配置
  rabbitmq:
    # 主机地址
    host: 192.168.136.103
    # 用户名、密码
    username: leyou
    password: leyou
    # 虚拟主机
    virtual-host: /leyou
```

#### 搜索微服务

##### 保存和删除索引

> 由于索引的新增和更新为同一个方法，所以只需要在`SearchService`中新增保存索引和删除索引的方法。

- 保存索引

  ```java
     /**
       * 新增或修改商品索引信息
       *
       * @param id 商品id
       */
      public void saveIndex(Long id) {
          // 查询商品
          SpuBO spuBO = goodsClient.queryGoodsById(id);
          Goods goods = buildGoods(spuBO);
  
          // 更新索引库
          goodsRepository.save(goods);
      }
  ```

- 删除索引

  ```java
      /**
       * 刪除商品索引
       *
       * @param id 商品id
       */
      public void deleteIndex(Long id) {
          goodsRepository.deleteById(id);
      }
  ```

##### 接收并处理消息

> 编写一个类，在其中定义两个方法，一个方法用于接收新增或更新商品的消息，另一个用于接收删除商品的消息。

```java
package com.leyou.search.listener;

import com.leyou.common.util.LeyouConstants;
import com.leyou.search.service.SearchService;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 处理商品微服务发送的消息
 */
@Component
public class GoodsListener {

    @Autowired
    private SearchService searchService;

    /**
     * 新增/更新 索引库
     *
     * @param id 商品id
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = LeyouConstants.QUEUE_SAVE_SEARCH, durable = "true"),
            exchange = @Exchange(name = LeyouConstants.EXCHANGE_DEFAULT_ITEM),
            key = {LeyouConstants.QUEUE_INSERT_ITEM, LeyouConstants.QUEUE_UPDATE_ITEM}
    ))
    public void saveIndex(Long id) {
        if (id != null)
            searchService.saveIndex(id);
    }

    /**
     * 删除指定商品的索引
     *
     * @param id 商品id
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = LeyouConstants.QUEUE_DELETE_SEARCH, durable = "true"),
            exchange = @Exchange(name = LeyouConstants.EXCHANGE_DEFAULT_ITEM),
            key = LeyouConstants.QUEUE_DELETE_ITEM
    ))
    public void deleteIndex(Long id) {
        if (id != null)
            searchService.deleteIndex(id);
    }

}
```

- 这里我没有指定交换机类型，采用默认的`Direct`。

- 在`LeyouConstants`中新增常量，如果觉得麻烦可以直接将字符串写死在注解上。

  ```java
      public static final String QUEUE_SAVE_SEARCH = "ly.search.save";
      public static final String QUEUE_DELETE_SEARCH = "ly.search.delete";
  ```

#### 商品详情页微服务

由于新增和修改商品详情页的方法已经在之前写好了，只需要新增删除商品详情页的方法。

##### 删除商品详情页

```java
    /**
     * 删除商品详情页
     *
     * @param id 商品id
     */
    public void deleteHtml(Long id) {
        File file = new File(pagePath + "/" + id + ".html");
        if (file.exists()) {
            // 删除文件
            file.delete();
        }
    }
```

##### 接收并处理消息

```java
package com.leyou.listener;

import com.leyou.common.util.LeyouConstants;
import com.leyou.page.service.PageService;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 处理商品微服务发送的消息
 */
@Component
public class GoodsListener {

    @Autowired
    private PageService pageService;

    /**
     * 新增/更新 商品详情页
     *
     * @param id 商品id
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = LeyouConstants.QUEUE_SAVE_PAGE, durable = "true"),
            exchange = @Exchange(name = LeyouConstants.EXCHANGE_DEFAULT_ITEM),
            key = {LeyouConstants.QUEUE_INSERT_ITEM, LeyouConstants.QUEUE_UPDATE_ITEM}
    ))
    public void savePage(Long id) {
        if (id != null)
            pageService.asyncExcute(id);
    }

    /**
     * 删除指定商品的详情页
     *
     * @param id 商品id
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = LeyouConstants.QUEUE_DELETE_PAGE, durable = "true"),
            exchange = @Exchange(name = LeyouConstants.EXCHANGE_DEFAULT_ITEM),
            key = LeyouConstants.QUEUE_DELETE_ITEM
    ))
    public void deleteIndex(Long id) {
        if (id != null)
            pageService.deleteHtml(id);
    }

}
```
