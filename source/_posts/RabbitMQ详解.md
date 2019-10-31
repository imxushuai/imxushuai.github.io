---
title: RabbitMQ详解
photo:
  - 'https://www.imxushuai.com/img/asset/rabbitmq.png'
tags:
  - RabbitMQ
  - RabbitMQ详解
  - 消息中间件
categories:
  - 消息中间件
  - RabbitMQ
description: 部署最广泛的开源消息中间件 - RabbitMQ
abbrlink: 30470
date: 2019-04-01 20:59:55
---

<center><i>部署最广泛的开源消息中间件 - RabbitMQ</i></center>

![](https://www.imxushuai.com/img/asset/rabbitmq.png)

<!-- more -->

说起`RabbitMQ`，首先就需要说起消息队列，那什么是消息队列呢？

### 什么是消息队列？

消息队列，即MQ，Message Queue。

消息队列是典型的：生产者、消费者模型。生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入，这样就实现了生产者和消费者的解耦。



### AMQP和JMS

MQ是消息通信的模型，并不是具体实现。现在实现MQ的有两种主流方式：AMQP、JMS。

#### AMQP

> **高级消息队列协议**即**Advanced Message Queuing Protocol**（AMQP）是一个用于统一[面向消息中间件](https://zh.wikipedia.org/w/index.php?title=%E9%9D%A2%E5%90%91%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6&action=edit&redlink=1)实现的一套标准协议，其设计目标是对于*消息*的排序、路由（包括点对点和[订阅-发布](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85)）、保持可靠性、保证安全性[[1\]](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E5%8D%8F%E8%AE%AE#cite_note-acmqueue-1)。高级消息队列协议保证了由不同提供商发行的客户端之间的[互操作性](https://zh.wikipedia.org/wiki/%E4%BA%92%E6%93%8D%E4%BD%9C%E6%80%A7)。与先前的中间件标准（如[Java消息服务](https://zh.wikipedia.org/wiki/Java%E6%B6%88%E6%81%AF%E6%9C%8D%E5%8A%A1)），在特定的API接口层面和实现行为进行了统一不同，高级消息队列协议关注于各种消息如何作为字节流进行传递。因此，使用了符合协议实现的任意应用程序之间可以保持对消息的创建、传递。

👆 摘自：[维基百科](https://zh.wikipedia.org/wiki/Wikipedia:%E9%A6%96%E9%A1%B5) 👆

#### JMS

> **Java消息服务**（**Java Message Service**，**JMS**）[应用程序接口](https://zh.wikipedia.org/wiki/%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E6%8E%A5%E5%8F%A3)是一个[Java](https://zh.wikipedia.org/wiki/Java)平台中关于[面向消息中间件](https://zh.wikipedia.org/w/index.php?title=%E9%9D%A2%E5%90%91%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6&action=edit&redlink=1)（MOM）的[API](https://zh.wikipedia.org/wiki/API)，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。
>
> Java消息服务的规范包括两种消息模式，点对点和发布者／订阅者。许多提供商支持这一通用框架因此，程序员可以在他们的分布式软件中实现面向消息的操作，这些操作将具有不同面向消息中间件产品的可移植性。
>
> Java消息服务支持同步和异步的消息处理，在某些场景下，同步消息是必要的；在其他场景下，异步消息比同步消息操作更加便利。
>
> Java消息服务支持面向事件的方法接收消息，[事件驱动的程序设计](https://zh.wikipedia.org/wiki/%E4%BA%8B%E4%BB%B6%E9%A9%85%E5%8B%95%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88)现在被广泛认为是一种富有成效的[程序设计范例](https://zh.wikipedia.org/wiki/%E7%BC%96%E7%A8%8B%E8%8C%83%E5%9E%8B)，程序员们都相当熟悉。
>
> 在应用系统开发时，Java消息服务可以推迟选择面对消息中间件产品，也可以在不同的面对消息中间件切换。

👆 摘自：[维基百科](https://zh.wikipedia.org/wiki/Wikipedia:%E9%A6%96%E9%A1%B5) 👆

#### 区别与联系

- JMS是定义了统一的接口，来对消息操作进行统一；AMQP是通过规定协议来统一数据交互的格式
- JMS限定了必须使用Java语言；AMQP只是协议，不规定实现方式，因此是跨语言的。
- JMS规定了两种消息模型；而AMQP的消息模型更加丰富



### 常见的消息队列产品

- ActiveMQ：基于JMS
- RabbitMQ：基于AMQP协议，erlang语言开发，稳定性好
- RocketMQ：基于JMS，阿里巴巴产品，目前交由Apache基金会
- Kafka：分布式消息系统，高吞吐量

> 此文章重点讲解的为采用AMQP的RabbitMQ



### RabbitMQ简介

> **RabbitMQ**是实现了[高级消息队列协议](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E5%8D%8F%E8%AE%AE)（AMQP）的开源[消息代理](https://zh.wikipedia.org/w/index.php?title=%E6%B6%88%E6%81%AF%E4%BB%A3%E7%90%86&action=edit&redlink=1)软件（亦称[面向消息的中间件](https://zh.wikipedia.org/w/index.php?title=%E9%9D%A2%E5%90%91%E6%B6%88%E6%81%AF%E7%9A%84%E4%B8%AD%E9%97%B4%E4%BB%B6&action=edit&redlink=1)）。RabbitMQ服务器是用[Erlang](https://zh.wikipedia.org/wiki/Erlang)语言编写的，而聚类和故障转移是构建在[开放电信平台](https://zh.wikipedia.org/wiki/%E9%96%8B%E6%94%BE%E9%9B%BB%E4%BF%A1%E5%B9%B3%E5%8F%B0)框架上的。所有主要的编程语言均有与代理接口通讯的客户端库。

👆 摘自：[维基百科](https://zh.wikipedia.org/wiki/Wikipedia:%E9%A6%96%E9%A1%B5) 👆

官网： https://www.rabbitmq.com/

官方教程：https://www.rabbitmq.com/getstarted.html

> 关于RabbitMQ的下载以及安装，请参考官方文档。

### RabbitMQ 基本概念

结构图

![](https://images.xushuai.fun/5015984-367dd717d89ae5db.png)

#### Message

 消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

#### Publisher

 消息的生产者，也是一个向交换器发布消息的客户端应用程序。

#### Exchange

 交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。在一般的消息中间件中会有三个角色，消息生产者/消息消费者以及消息的队列，而在rabbitmq中多出一个角色，就是交换器。消息生产者先将消息交给交换器，然后交换器根据不同路由策略将消息传送到不同的消息队列中。

#### Binding

 绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

#### Queue
 消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

#### Connection
 网络连接，比如一个TCP连接。

#### Channel
 信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

#### Consumer
 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

#### Virtual Host
 虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

#### Broker
 表示消息队列服务器实体。

### 代码获取

文章中所有代码都已经上传至`Github`，可以通过下方的链接下载代码。

[https://github.com](https://github.com/imxushuai/Demo-Repository/tree/master/rabbitmq-demo)

### RabbitMQ Java Api使用

![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qrde11wcj30s50hlq4r.jpg)

> 上面这是官方的教程，可以看到分为了6个模块，下面我会对其中的前5种进行编码，关于RPC可以到官网查阅文档。当然如果你阅读文档的能力足够，建议直接前往官网，可能收获会更多。

#### Hello World

##### 说明

RabbitMQ是一个消息代理：它接受和转发消息。 你可以把它想象成一个邮局：当你把邮件放在邮箱里时，你可以确定邮差先生最终会把邮件发送给你的收件人。 在这个比喻中，RabbitMQ是邮政信箱，邮局和邮递员。

![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qt642cnfj30aw01nwec.jpg)

P（producer/ publisher）：生产者，一个发送消息的用户应用程序。

C（consumer）：消费者，消费和接收有类似的意思，消费者是一个主要用来等待接收消息的用户应用程序。

队列（红色区域）：rabbitmq内部类似于邮箱的一个概念。虽然消息流经rabbitmq和你的应用程序，但是它们只能存储在队列中。队列只受主机的内存和磁盘限制，实质上是一个大的消息缓冲区。许多生产者可以发送消息到一个队列，许多消费者可以尝试从一个队列接收数据。

##### 生产者

```java
package fun.xushuai.rabbitmq.nativeapi.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * 消息生产者
 */
public class SimpleProducer {
    private final static String QUEUE_NAME = "Simple_Queue";

    public static void main(String[] args) {
        // 创建连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.136.103");
        /* 下面这些值可以不用设置，都可以使用默认值(前提：未作特殊配置的话)
            //端口
            factory.setPort(5672);
            //设置账号信息，用户名、密码、虚拟主机
            factory.setVirtualHost("/");
            factory.setUsername("guest");
            factory.setPassword("guest");
        */
        try {
            Connection connection = factory.newConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            //发送消息
            String message = "Hello Rabbit!!! time = " + System.currentTimeMillis();
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());

            System.out.println(" [x] Sent '" + message + "'");
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

##### 消费者

```java
package fun.xushuai.rabbitmq.nativeapi.simple;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * 消息消费者
 */
public class SimpleConsumer {
    private final static String QUEUE_NAME = "Simple_Queue";

    public static void main(String[] args) {
        // 创建连接
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.136.103");
        Connection connection = null;
        try {
            connection = factory.newConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);

            // 获取消息
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [x] Received '" + message + "'");
            };
            channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {});
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

##### 测试

1. 运行生产者

   ![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qtssaj6jj30e601cdfp.jpg)

2. 运行消费者

![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qttv5zxej30dx01awec.jpg)

> 成功拿到消息

##### 消息确认机制(ACK)

在订阅消息的时候可以指定应答模式,当自动应答等于true的时候，表示当消费者一收到消息就表示消费者收到了消息，消费者收到了消息就会立即从队列中删除。

这样就会存在一个问题，如果我在拿到消息后执行逻辑时候出现了异常，此时这条消息已经在队列中被删除了，如果此消息非常的重要，那么就会造成不可挽回的错误。

因此，RabbitMQ有一个ACK机制。当消费者获取消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。不过这种回执ACK分两种情况：

- 自动ACK：消息一旦被接收，消费者自动发送ACK
- 手动ACK：消息接收后，不会发送ACK，需要手动调用

在之前的代码中使用的是自动ACK，现在我们修改为手动ACK，改动如下：

![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2quegi5lgj30nw0dfmy5.jpg)

> 我在应答之前抛出了异常，这个消息就不会被删除，反复重启消费者，就会发现会一直拿到这个消息。

#### Work(竞争消费者模式)

##### 说明

工作队列，又称任务队列。主要思想就是避免执行资源密集型任务时，必须等待它执行完成。相反我们稍后完成任务，我们将任务封装为消息并将其发送到队列。 在后台运行的工作进程将获取任务并最终执行作业。当你运行许多工人时，任务将在他们之间共享，但是一个消息只能被一个消费者获取。

![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qut29j13j3098033a9z.jpg)

#####  生产者

```java
package fun.xushuai.rabbitmq.nativeapi.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.io.IOException;

/**
 * Work(竞争消费者模式) - 生产者
 */
public class WorkProducer {
    private final static String QUEUE_NAME = "Work_Queue";

    public static void main(String[] args) {
        try {
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            doWork(channel);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void doWork(Channel channel) throws IOException {
        // 循环发送消息，模拟并发场景
        for (int i = 0; i < 20; i++) {
            //发送消息
            String message = i + " : 发送消息";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        }
    }
}
```

##### 消费者

```java
package fun.xushuai.rabbitmq.nativeapi.work;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.DeliverCallback;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * Work(竞争消费者模式) - 消费者
 */
public class WorkConsumer {
    private final static String QUEUE_NAME = "Work_Queue";

    public static void main(String[] args) {
        try {
            Connection connection = ConnectionUtil.getConnection();
            Channel channel = connection.createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [x] Received '" + message + "'");
                //模拟耗时
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException ignored) {
                }
                // 消息应答
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            };
            // 第二个参数改为false，表示不启用自动应答
            channel.basicConsume(QUEUE_NAME, false, deliverCallback, consumerTag -> {});
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

##### 测试

> 测试Work必须先运行消费者且还要运行多个，我这里运行两个，有测试效果就行。

1. 先运行一个消费者

2. 将消费者中 '模拟耗时' 部分注释掉，再复制一个运行，操作步骤如下：

   > 也可以直接新建第二个消费者类，简单粗暴。

   - 注释 '模拟耗时' 部分代码

     ![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qw4kvp7ej30q20cd0tm.jpg)

   - 进入运行配置界面

     ![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qvvzo47aj30qg081t99.jpg)

   - 复制WorkConsumer

     ![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qw0jbh72j30wo0j9abd.jpg)

   - 运行复制出来的WorkConsumer

     ![](https://www.imxushuai.com/img/asset/006ifTg0gy1g2qw1lg79jj30c405kaa0.jpg)

3. 观察运行结果，会发现有无模拟耗时执行的任务数是一致的，像是消费者轮询消费消息。

   ![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rlp00h5jj309j08pwgf.jpg)![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rlp00td9j309p08fmz4.jpg)

   > 原因：RabbitMQ 只管分发进入队列的消息，不会关心有多少消费者（consumer）没有作出响应。它盲目的把消息分发给多个消费者，直到队列中无新消息。

##### 更改调度模式

在上面的测试中，发现最终的结果并没有达到预期，预期的效果是效率较高的消费者应该执行更多的任务，而不是平均分配。

我们可以使用 basic.qos 方法，并设置 prefetch_count=1。这样是告诉RabbitMQ，再同一时刻，不要发送超过1条消息给一个工作者（worker），直到它已经处理了上一条消息并且作出了响应。这样，RabbitMQ 就会把消息分发给下一个空闲的工作者（worker）。

修改`WorkConsumer.java`，添加代码：

![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rmca4zjzj30lo08wjuu.jpg)

重启测试，会发现耗时较快的消费者，执行了更多的任务。

![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rmfzdsxwj30a809zwf9.jpg)![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rmfzdsv7j30b50a1q5p.jpg)

#### Publish/Subscribe(发布/订阅)

##### 说明

在Work模式背后的假设是每个任务都交付给一个工作者。在这一部分，我们将做一些完全不同的事情 - 我们将向多个消费者传递信息，此模式称为“发布/订阅”。

![](https://www.imxushuai.com/img/asset/006kPtiPgy1g2rmvbxzjrj30c2033q2x.jpg)

##### 特征

1. 1个生产者，多个消费者

2. 每一个消费者都有自己的一个队列

3. 生产者没有将消息直接发送到队列，而是发送到了交换机

4. 每个队列都要绑定到交换机

5. 生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者获取的目的

X（Exchanges）：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

##### Exchange类型

- Fanout：广播，将消息交给所有绑定到交换机的队列

- Direct：定向，把消息交给符合指定routing key 的队列

- Topics：通配符，把消息交给符合routing pattern（路由模式） 的队列

在官方教程中对应的后三种，其实都是`Publish/Subscribe`，只是`Exchange`的类型不同，做了区分。

#### Fanout(广播)

##### 说明

在广播模式下，消息发送流程是这样的：

-  可以有多个消费者
- 每个**消费者有自己的queue**（队列）
- 每个**队列都要绑定到Exchange**（交换机）
- **生产者发送的消息，只能发送到交换机**，交换机来决定要发给哪个队列，生产者无法决定。
- 交换机把消息发送给绑定过的所有队列
- 队列的消费者都能拿到消息。实现一条消息被多个消费者消费
- 若交换机收到消息时，没有队列与其绑定，消息将丢失

由于获取连接的代码是重复的，所以抽取了一个工具类

```java
package fun.xushuai.rabbitmq.nativeapi.util;

import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;

public final class ConnectionUtil {

    /**
     * 建立与RabbitMQ的连接
     */
    public static Connection getConnection() throws Exception {
        //定义连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置服务地址
        factory.setHost("192.168.136.103");
        //端口
        factory.setPort(5672);
        //设置账号信息，用户名、密码、虚拟主机
        factory.setVirtualHost("/");
        factory.setUsername("guest");
        factory.setPassword("guest");
        // 通过工程获取连接
        return factory.newConnection();
    }

}
```

##### 生产者

```java
package fun.xushuai.rabbitmq.nativeapi.fanout;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Fanout
 * 生产者
 */
public class FanoutProducer {
    private final static String EXCHANGE_NAME = "Fanout_Exchange";

    public static void main(String[] args) {
        try {
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            // 声明 Exchange
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);

            // 发送消息到 Exchange
            String message = "[Fanout] Sent Message to Exchange";
            channel.basicPublish(EXCHANGE_NAME, "",null, message.getBytes(StandardCharsets.UTF_8));

            System.out.println(" [Fanout] Sent '" + message + "'");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 使用`exchangeDeclare(exchangeName, exchangeType)`声明`Exchange`
- 发送消息到`Exchange`，使用的是第一个参数，之前的`Hello World`和`Work`都是使用的第二个参数

##### 消费者

```java
package fun.xushuai.rabbitmq.nativeapi.fanout;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.DeliverCallback;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Fanout
 * 消费者
 */
public class FanoutConsumer {
    private final static String EXCHANGE_NAME = "Fanout_Exchange";

    public static void main(String[] args) {
        try {
            // 获取连接
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
            String queueName = channel.queueDeclare().getQueue();
            // 绑定 Exchange
            channel.queueBind(queueName, EXCHANGE_NAME, "");
            System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

            // 获取消息
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [x] Received '" + message + "'");
                // 消息应答
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            };
            // 第二个参数改为false，表示不启用自动应答
            channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 从`queueDeclare`中获取`queueName`，并将该队列绑定到`Exchange`

##### 测试

> 同样运行两个消费者实例，可以像Work模式一样复制一个运行或者直接创建第二个消费者类运行。**需要先启动两个Consumer**。

最终的运行结果，会发现两个`Consumer`都可以收到生产者发出的消息。这就是`发布/订阅`中的`Fanout`模式。

#### Direct(定向)

##### 说明

有选择性的接收消息，也就是官方教程中的`Routing`。

在订阅模式中，生产者发布消息，所有消费者都可以获取所有消息。

在路由模式中，我们将添加一个功能 - 我们将只能订阅一部分消息。 例如，我们只能将重要的错误消息引导到日志文件（以节省磁盘空间），同时仍然能够在控制台上打印所有日志消息。

但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

在Direct模型下，队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

消息的发送方在向Exchange发送消息时，也必须指定消息的routing key。

![](https://www.imxushuai.com/img/asset/direct-exchange.png)

P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key 为 orange 的消息

C2：消费者，其所在队列指定了需要routing key 为 black、green 的消息

##### 生产者

```java
package fun.xushuai.rabbitmq.nativeapi.direct;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Direct (Routing)
 * 生产者
 */
public class DirectProducer {
    private final static String EXCHANGE_NAME = "Direct_Exchange";

    private final static String[] ROUTING_KEYS = new String[]{"orange", "black", "green"};

    public static void main(String[] args) {
        try {
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            // 声明 Exchange
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            // 发送消息到 Exchange
            String message = "[Direct] Sent Message to ";
            for (String routingKey : ROUTING_KEYS) {
                channel.basicPublish(EXCHANGE_NAME, routingKey,null, (message + routingKey).getBytes(StandardCharsets.UTF_8));
                System.out.println(" [Direct] Sent '" + message + routingKey + "'");
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

##### 消费者-1

该消费者，消费routingKey为black和green的消息

```java
package fun.xushuai.rabbitmq.nativeapi.direct;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.DeliverCallback;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Direct (Routing)
 * 消费者
 */
public class DirectConsumer {
    private final static String EXCHANGE_NAME = "Direct_Exchange";

    private final static String ROUTING_KEY_BLACK = "black";
    private final static String ROUTING_KEY_GREEN = "green";

    public static void main(String[] args) {
        try {
            // 获取连接
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
            String queueName = channel.queueDeclare().getQueue();
            // 绑定 Exchange 并且指定routingKey
            channel.queueBind(queueName, EXCHANGE_NAME, ROUTING_KEY_BLACK);
            channel.queueBind(queueName, EXCHANGE_NAME, ROUTING_KEY_GREEN);

            System.out.println("[消费者1] RoutingKey = [black, green]");

            // 获取消息
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [消费者1] 收到消息, message= [" + message + "]");
                // 消息应答
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            };
            // 第二个参数改为false，表示不启用自动应答
            channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

##### 消费者-2

该消费者消费routingKey为orange的消息

```java
package fun.xushuai.rabbitmq.nativeapi.direct;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.DeliverCallback;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Direct (Routing)
 * 消费者
 */
public class DirectConsumer2 {
    private final static String EXCHANGE_NAME = "Direct_Exchange";

    private final static String ROUTING_KEY_ORANGE = "orange";

    public static void main(String[] args) {
        try {
            // 获取连接
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
            String queueName = channel.queueDeclare().getQueue();
            // 绑定 Exchange 并且指定routingKey
            channel.queueBind(queueName, EXCHANGE_NAME, ROUTING_KEY_ORANGE);

            System.out.println("[消费者2] RoutingKey = [orange]");

            // 获取消息
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [消费者2] 收到消息, message= [" + message + "]");
                // 消息应答
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            };
            // 第二个参数改为false，表示不启用自动应答
            channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

##### 测试

运行消费者，然后运行生产者。

会发现只有`消费者-1`接收到了消息，即绑定了`black`的消费者收到了消息，可以尝试修改生产者发送到`orange`的`routingKey`，会发现只有`消费者-2`收到了 消息，这样就可以完成，不同的消息，发送给不同的消费者。

#### Topic(匹配)

##### 说明

`Topic`类型的`Exchange`与`Direct`相比，都是可以根据`RoutingKey`把消息路由到不同的队列。只不过`Topic`类型`Exchange`可以让队列在绑定`Routing key` 的时候使用通配符！

`Routingkey` 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： `item.insert`

 通配符规则：

​         `#`：匹配一个或多个词

​         `*`：匹配不多不少恰好1个词

![](https://www.imxushuai.com/img/asset/20150914161921517.jpg)

- 以`usa`开头的消息会进入第一个队列
- 以`news`结尾的会进入第二个队列
- 以`weather`结尾的会进入第三个队列
- 以`europe`开发的会进入第四个队列

下面我们就来模拟这个案例

##### 生产者

```java
package fun.xushuai.rabbitmq.nativeapi.topics;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Topic
 * 生产者
 */
public class TopicsProducer {
    private final static String EXCHANGE_NAME = "Topics_Exchange";

    private final static String[] routingKeys = new String[]{"usa.news", "usa.weather", "europe.news", "europe.weather"};

    public static void main(String[] args) {
        try {
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            // 声明 Exchange
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

            sendMessage(channel);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void sendMessage(Channel channel) throws IOException {
        for (String routingKey : routingKeys) {
            // 发送消息到 Exchange
            String message = "[Topics] " + routingKey;
            // 指定RoutingKey为 'black'
            channel.basicPublish(EXCHANGE_NAME, routingKey,null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println("[Topics] message = " + routingKey);
        }
    }
}
```

##### 消费者

```java
package fun.xushuai.rabbitmq.nativeapi.topics;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.DeliverCallback;
import fun.xushuai.rabbitmq.nativeapi.util.ConnectionUtil;

import java.nio.charset.StandardCharsets;

/**
 * 发布/订阅模型 - Topic
 * 消费者
 */
public class TopicsConsumer1 {
    private final static String EXCHANGE_NAME = "Topics_Exchange";

    private final static String ROUTING_KEY_START_WITH_USA = "usa.#";

    public static void main(String[] args) {
        try {
            // 获取连接
            Connection connection = ConnectionUtil.getConnection();
            // 声明队列，存在不做改变，不存在则创建topic
            Channel channel = connection.createChannel();
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
            String queueName = channel.queueDeclare().getQueue();
            // 绑定 Exchange 并且指定routingKey
            channel.queueBind(queueName, EXCHANGE_NAME, ROUTING_KEY_START_WITH_USA);

            System.out.println("[消费者1] RoutingKey = [usa.#]");

            // 获取消息
            DeliverCallback deliverCallback = (consumerTag, delivery) -> {
                // 获取消息
                String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
                System.out.println(" [消费者1] 收到消息, message= [" + message + "]");
                // 消息应答
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            };
            // 第二个参数改为false，表示不启用自动应答
            channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

总共需要四个消费者，`change.queueBind`分别绑定`usa.#`，`#.news`,`#.weather`,`europe.#`

##### 测试

先运行四个消费者，然后运行生产者，查看消费者接收消息的情况。

- `usa.#`：接收到了 `usa.news` 和 `usa.weather`

  ![](https://www.imxushuai.com/img/asset/20190507231554.png)

- `#.news`：接收到了`usa.news`和`europe.news`

  ![](https://www.imxushuai.com/img/asset/20190507231610.png)

- `#.weather`：接收到了`usa.weather`和`europe.weather`

  ![](https://www.imxushuai.com/img/asset/20190507231619.png)

- `europe.#`：接收到了`europe.news`和`europe.weather`

  ![](https://www.imxushuai.com/img/asset/20190507231629.png)

#### 扩展：消息持久化

在之前，如果在拿到消息后执行过程中发生了错误，可以使用`RabbitMQ`自带的`ACK机制`去避免消息的丢失，若是在消息发送到`RabbitMQ`后，消费者拿到消息之前，`RabbitMQ`宕机也会造成消息的丢失，那这种情况该怎么办呢？

这就需要使用到`RabbitMQ`自带的持久化。

RabbitMQ中的消息持久化对象是：

- 消息

- 消息队列
- Exchange

下面就介绍如何对上面三种对象进行持久化操作

##### 持久化 - 消息

```java
    /**
     * 发布消息
     *
     * 发送消息到不存在的交换机将导致异常
     * 发生异常时，通道会被关闭
     *
     * @param 需要将消息发送到的交换机名
     * @param routingKey
     * @param 消息头
     * @param 消息体
     */
    void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;
```

这里消息的持久化需要设置的参数为`props`，参数类型为`BasicProperties`

`BasicProperties`详解

在 *AMQP* 协议中，为消息预定了 14 个属性，如下：

- *content_type*：标明消息的类型.
- *content_encoding*：标明消息的编码.
- *headers*：可扩展的信息对.
- *delivery_mode*：为 `2` 时表示该消息需要被持久化支持.
- *priority*：该消息的权重.
- *correlation_id*：用于"请求"与"响应"之间的匹配.
- *reply_to*："响应"的目标队列.
- *expiration*：有效期.
- *message_id*：消息的ID.
- *timestamp*：一个时间戳.
- *type*：消息的类型.
- *user_id*：用户的ID.
- *app_id*：应用的ID.
- *cluster_id*：服务集群ID.

属性这么多，那到底该怎么设置呢？

在`RabiitMQ`的`MessageProperties`类中进行了一些六个`BasicProperties`配置

```java
package com.rabbitmq.client;

import com.rabbitmq.client.AMQP.BasicProperties;
import com.rabbitmq.client.impl.AMQContentHeader;

/**
 * 常量holder类，包含AMQContentHeader的有用静态实例。 
 * 这些用于 basicPublish和其他Channel方法。
 */
public class MessageProperties {

    /** 空基本属性，未设置字段 */
    public static final BasicProperties MINIMAL_BASIC =
        new BasicProperties(null, null, null, null,null, null, null, null,
                            null, null, null, null,null, null);
    /** 空基本属性，仅将deliveryMode设置为2（持久性） */
    public static final BasicProperties MINIMAL_PERSISTENT_BASIC =
        new BasicProperties(null, null, null, 2,null, null, null, null,
                            null, null, null, null,null, null);

    /** 内容类型“application/octet-stream”，deliveryMode 1（非持久），优先级为零 */
    public static final BasicProperties BASIC =
        new BasicProperties("application/octet-stream", null, null, 1,
                            0, null, null, null,null, null, null, null,
                            null, null);

    /** 内容类型“application/octet-stream”，deliveryMode 2（持久性），优先级为零 */
    public static final BasicProperties PERSISTENT_BASIC =
        new BasicProperties("application/octet-stream",null,null,2,
                            0, null, null, null, null, null, null, null,
                            null, null);

    /** 内容类型“text/plain”，deliveryMode 1（非持久性），优先级为零 */
    public static final BasicProperties TEXT_PLAIN =
        new BasicProperties("text/plain", null, null, 1,
                            0, null, null, null, null, null, null, null,
                            null, null);

    /** 内容类型“text/plain”，deliveryMode 2（持久性），优先级为零 */
    public static final BasicProperties PERSISTENT_TEXT_PLAIN =
        new BasicProperties("text/plain", null, null, 2,
                            0, null, null, null, null, null, null, null,
                            null, null);
}

```

根据传递的信息选择，这里我们传输的消息主要为文本消息，所以使用`PERSISTENT_TEXT_PLAIN`即可。

示例：

```java
channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, "message".getBytes());
```

##### 持久化 - 消息队列

```java
    /**
     * 声明队列
     * @param 队列名称
     * @param 是否持久化队列，如果为true，队列在服务器重启后继续存在
     * @param 是否为独占队列（连接层面的独占）
     * @param 是否自动删除，为true，队列不再使用时被自动删除
     * @param 其他参数
     */
    Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException;
```

> 截取自`RabbitMQ`源码

在生命队列时，设置持久化参数`durable`为`true`。

示例：

```java
channel.queueDeclare("queueName", true, false, false, null);
```

##### 持久化 - Exchange

```java
    /**
     * 声明交换机
     * @param 交换机名称
     * @param 交换机类型
     * @param 是否持久化交换机，如果为true，交换将在服务器重启后继续存在
     */
    Exchange.DeclareOk exchangeDeclare(String exchange, String type, boolean durable) throws IOException;
```

>  截取自`RabbitMQ`源码

所以只需要在声明交换机时，设置持久化参数为`true`即可。

示例：

```java
channel.exchangeDeclare("exchangeName", BuiltinExchangeType.FANOUT, true);
```

### Spring AMQP

`Spring AMQP`是对AMQP协议的抽象实现，而`Spring Rabbit` 是对协议的具体实现。

[Spring AMQP官网](<https://spring.io/projects/spring-amqp>)

话不多说直接上代码！

##### Spring boot + RabbitMQ

###### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

###### Rabbit MQ配置

```yaml
spring:
  rabbitmq:
    # 安装rabbitmq的主机地址
    host: 192.168.136.103
    # rabbitmq的用户名，安装后默认有一个用户，账号密码都是：guest
    username: guest
    password: guest
    # 安装好后guest账号默认的虚拟主机
    virtual-host: /

rabbitmq:
  # 将队列和交换机名称放入配置文件方便管理
  simpleQueue: simpleQueue
  simpleExchange: simpleExchange
```

###### Spring boot 启动类

```java
package fun.xushuai.rabbitmq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class RabbitmqApplication {
    public static void main(String[] args) {
        SpringApplication.run(RabbitmqApplication.class, args);
    }
}
```

> 我这里用到了定时任务，所以使用`@EnableScheduling`开启定时任务支持。

###### RabbitMQ配置类

主要用于配置队列，交换机等内容

```java
package fun.xushuai.rabbitmq.springamqp.config;

import org.springframework.amqp.core.Exchange;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * RabbitMQ配置类
 */
@Configuration
public class RabbitMQConfiguration {

    @Value("${rabbitmq.simpleQueue}")
    public String simpleQueue;

    @Value("${rabbitmq.simpleExchange}")
    public String simpleExchange;

    /**
     * 声明队列
     */
    @Bean
    public Queue simpleQueue() {
        return new Queue(simpleQueue, true);
    }

    /**
     * 声明交换器(Exchange)
     *
     * Exchange是个接口，拥有六个实现类，分别是：
     * AbstractExchange(抽象实现), CustomExchange, DirectExchange
     * FanoutExchange, HeadersExchange, TopicExchange
     */
    @Bean
    public Exchange simpleExchange() {
        return new FanoutExchange(simpleExchange, true, false);
    }
}
```

###### 生产者

```java
package fun.xushuai.rabbitmq.springamqp.simple;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * Spring amqp的简单使用
 * 生产者
 */
@Component
public class SpringAMQPProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Value("${rabbitmq.simpleQueue}")
    public String simpleQueue;


    @Value("${rabbitmq.simpleExchange}")
    public String simpleExchange;

    /**
     * 定时发送消息
     * 每隔十秒发送一条消息
     */
    @Scheduled(fixedRate = 10000)
    public void sendMessageToQueue() {
        String msg = "hello spring amqp! 现在时间是:" + System.currentTimeMillis();
        rabbitTemplate.convertAndSend(simpleQueue, msg);
        System.out.println("发送消息到 simpleQueue, message = " + msg);
    }

    /**
     * 定时发送消息
     * 每隔十秒发送一条消息
     */
    @Scheduled(fixedRate = 10000)
    public void sendMessageToExchange() {
        String msg = "hello spring amqp! 现在时间是:" + System.currentTimeMillis();
        // 发送消息到 simpleExchange
        rabbitTemplate.convertAndSend(simpleExchange, "", msg);
        System.out.println("发送消息到 simpleExchange, message = " + msg);
    }
}

```

###### 消费者

- 第一种方式，将`@RabbitListener`注解放在类上，然后使用`@RabbitHandler`注解在方法上进行消息的接收和处理。

  ```java
  package fun.xushuai.rabbitmq.springamqp.simple;
  
  import org.springframework.amqp.rabbit.annotation.RabbitHandler;
  import org.springframework.amqp.rabbit.annotation.RabbitListener;
  import org.springframework.stereotype.Component;
  
  /**
   * Spring amqp的简单使用
   * 生产者
   * 若要该类生效，需要打开 @Component 的注解
   */
  //@Component
  @RabbitListener(queues = "simpleQueue")
  public class SpringAMQPConsumer {
  
      @RabbitHandler
      public void process(String msg) {
          System.out.println("[消费者1] simpleQueue 收到消息：" + msg);
      }
  }
  
  ```

- 第二种方式，直接将`@RabbitLisener`注解使用在方法上。

  ```java
  package fun.xushuai.rabbitmq.springamqp.simple;
  
  import org.springframework.amqp.core.ExchangeTypes;
  import org.springframework.amqp.rabbit.annotation.Exchange;
  import org.springframework.amqp.rabbit.annotation.Queue;
  import org.springframework.amqp.rabbit.annotation.QueueBinding;
  import org.springframework.amqp.rabbit.annotation.RabbitListener;
  import org.springframework.stereotype.Component;
  
  @Component
  public class SpringAMQPConsumer2 {
  
      @RabbitListener(queues = "simpleQueue")
      private void process(String msg) {
          System.out.println("[消费者2#process] simpleQueue 收到消息：" + msg);
      }
  
      @RabbitListener(bindings = @QueueBinding(
              value = @Queue(),
              exchange = @Exchange(name = "simpleExchange", type = ExchangeTypes.FANOUT)))
      public void processMessageFromExchange1(String msg) {
          System.out.println("[消费者2#processMessageFromExchange1] simpleExchange 收到消息：" + msg);
      }
  
      @RabbitListener(bindings = @QueueBinding(
              value = @Queue(),
              exchange = @Exchange(name = "simpleExchange", type = ExchangeTypes.FANOUT)))
      public void processMessageFromExchange2(String msg) {
          System.out.println("[消费者2#processMessageFromExchange2] simpleExchange 收到消息：" + msg);
      }
  }
  ```

  这里对第二种方法中的注解做简要说明，主要对`bindings`中的注解

  - `@RabbitListener`：即可声明在方法上，也可以声明在类上，标识方法或类为消费者。
  - `@QueueBinding`：设置消费者的绑定信息
    1. value：使用`@Queue`指定绑定的队列，`@Queue`的值主要需要设置队列的名称
    2. exchange：使用`@Exchange`绑定到交换器，主要需要设置的是交换器的名称，交换器类型(默认为Direct)
    3. key：若需要指定`RoutingKey`，则在`key`中定义

###### 测试

![](https://www.imxushuai.com/img/asset/20190508193104.png)

> 发送到队列`simpleQueue`的消息，顺利被消费。
>
> 已`Fanout(广播)`发送到`simpleExchange`的消息也被两个消费者拿到。

