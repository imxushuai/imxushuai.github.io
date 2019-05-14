---
title: Elasticsearch的介绍和安装
photo:
  - 'https://images.xushuai.fun/elasticsearch-logo.png'
date: 2019-03-29 15:13:52
tags:
  - elasticsearch
  - 全文检索
categories:
  - 全文检索
  - elasticsearch
description: 全文检索引擎之Elasticsearch的简介及安装
---

<center><i>全文检索引擎之Elasticsearch的简介及安装</i></center>

![](https://images.xushuai.fun/elasticsearch-logo.png)

<!-- more -->

## Elasticsearch介绍

> **Elasticsearch**是一个基于[Lucene](https://zh.wikipedia.org/wiki/Lucene)库的[搜索引擎](https://zh.wikipedia.org/wiki/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E)。它提供了一个分布式、支持多租户的[全文搜索](https://zh.wikipedia.org/wiki/%E5%85%A8%E6%96%87%E6%AA%A2%E7%B4%A2)引擎，具有[HTTP](https://zh.wikipedia.org/wiki/HTTP) Web接口和无模式[JSON](https://zh.wikipedia.org/wiki/JSON)文档。Elasticsearch是用[Java](https://zh.wikipedia.org/wiki/Java)开发的，并在[Apache许可证](https://zh.wikipedia.org/wiki/Apache%E8%AE%B8%E5%8F%AF%E8%AF%81)下作为开源软件发布。官方客户端在[Java](https://zh.wikipedia.org/wiki/Java)、[.NET](https://zh.wikipedia.org/wiki/.NET%E6%A1%86%E6%9E%B6)（[C#](https://zh.wikipedia.org/wiki/C%E2%99%AF)）、[PHP](https://zh.wikipedia.org/wiki/PHP)、[Python](https://zh.wikipedia.org/wiki/Python)、[Apache Groovy](https://zh.wikipedia.org/wiki/Groovy)、[Ruby](https://zh.wikipedia.org/wiki/Ruby)和许多其他语言中都是可用的。[[5\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-offizsite-5)根据DB-Engines的排名显示，Elasticsearch是最受欢迎的企业搜索引擎，其次是[Apache Solr](https://zh.wikipedia.org/wiki/Apache_Solr)，也是基于Lucene。[[6\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-6)
>
> Elasticsearch是与名为Logstash的数据收集和日志解析引擎以及名为Kibana的分析和可视化平台一起开发的。这三个产品被设计成一个集成解决方案，称为“Elastic Stack”（以前称为“ELK stack”）。
>
> Elasticsearch可以用于搜索各种文档。它提供可扩展的搜索，具有接近实时的搜索，并支持多租户。[[5\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-offizsite-5)”Elasticsearch是分布式的，这意味着索引可以被分成分片，每个分片可以有0个或多个副本。每个节点托管一个或多个分片，并充当协调器将操作委托给正确的分片。再平衡和路由是自动完成的。“[[5\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-offizsite-5)相关数据通常存储在同一个索引中，该索引由一个或多个主分片和零个或多个复制分片组成。一旦创建了索引，就不能更改主分片的数量。[[7\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-7)
>
> Elasticsearch使用Lucene，并试图通过JSON和Java API提供其所有特性。它支持facetting和percolating[[8\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-8)，如果新文档与注册查询匹配，这对于通知非常有用。
>
> 另一个特性称为“网关”，处理索引的长期持久性；例如，在服务器崩溃的情况下，可以从网关恢复索引。[[9\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-gateway-9)Elasticsearch支持实时GET请求，适合作为[NoSQL](https://zh.wikipedia.org/wiki/NoSQL)数据存储[[10\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-jetslidedatabase-10)，但缺少分布式事务。[[11\]](https://zh.wikipedia.org/wiki/Elasticsearch#cite_note-transactions-11)

👆 摘自：[维基百科](https://zh.wikipedia.org/wiki/Wikipedia:%E9%A6%96%E9%A1%B5) 👆

[Elasticsearch官网](https://www.elastic.co/cn/products/elasticsearch)



## Elasticsearch安装

### 准备工作

> 我使用的Centos7的虚拟机进行安装

- 安装好jdk8的虚拟机一台（注意配置 `JAVA_HOME`）
- 下载`Elasticsearch`压缩包（我这里使用的版本是 6.6.1）

### 安装

1. Elasticsearch的安装非常的简单，仅仅只需要解压下载好的压缩包即可

> 注意:
>
> ​	请勿使用root用户安装，Elasticsearch考虑到安全问题，使用root用户运行会报错。所以，在解压之前需要单独创建一个用户，然后登陆创建的用户解压即可。

### 配置

> Elasticsearch的安装非常简单，但是要运行，还需要一点小小的配置，配置文件位于安装目录中的`config`目录中。

- Elastci的配置文件分为三个：
  - `elasticsearch.yml` 用于配置Elasticsearch
  - `jvm.options`用于配置Elasticsearch JVM设置
  - `log4j2.properties 用于配置Elasticsearch日志记录`

[Elsticsearch重要配置参考](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)

1. 配置`jvm.options`

   ```config
   # Xms 最小堆大小，默认1g
   # Xmx 最大堆大小，默认1g
   -Xms256m
   -Xmx256m
   ```

   > 这些设置的值取决于服务器上可用的RAM量，可以参考官方的文档进行配置，地址为上面给出的配置参考链接。

2. 配置`elasticsearch.yml`

   ```yaml
   # 配置path.log和path.data，存放数据文件和日志文件的目录
   # 目录具体位置以你的安装目录为准
   path.data: /home/xushuai/elasticsearch-6.6.1/data
   path.logs: /home/xushuai/elasticsearch-6.6.1/logs
   # 配置集群名称，集群唯一标识，作用类似于ID
   # 若不配置，默认为：elasticsearch
   cluster.name: my-elasticsearch
   # 配置节点名称，在集群中唯一表示当前节点
   node.name: node-1
   # 通配IP访问
   network.host: 0.0.0.0
   ## ......还有一些集群配置，可以查看官方文档
   ```

### 启动测试

1. 执行安装目录的中的`bin/elasticsearch`运行elasticsearch

   - `bin/elasticsearch -d`为后台运行

2. 访问测试

   ![](https://images.xushuai.fun/20190227001100.png)

### 启动错误处理

> 启动失败后，注意查看日志

1. max number of threads [3795] for user [xushuai] is too low, increase to at least [4096]

   - 修改 `/etc/security/limits.d/`下的`xx-nproc.conf`文件，具体文件名以你的服务器为准。

     ```conf
     *          soft    nproc     4096
     ```

     

2. max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

   - 修改`/etc/security/limits.conf`文件

     ```conf
     # 添加配置
     *               soft    nproc           65536
     *               hard    nproc           65536
     *               soft    nofile          65536
     *               hard    nofile          65536
     ```

     

3.  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

   - 修改`/etc/sysctl.conf `文件

     ```conf
     vm.max_map_count=262144
     ```

   - 保存后，执行命令 `sysctl -p`

> 配置修改完毕后，仍无法启动成功，请重启虚拟机。
>
> 正常启动后，如无法从web访问到，请注意查看防火墙是否关闭。



## 安装Kibana作为可视化平台

[Kibana官网](https://www.elastic.co/cn/products/kibana)

### Kibana介绍

> Kibana是Elasticsearch的开源数据可视化插件。它在Elasticsearch集群上索引的内容之上提供可视化功能。用户可以在大量数据之上创建条形图，折线图和散点图，或饼图和贴图。 Elasticsearch，Logstash和Kibana的组合，称为“弹性堆栈”，可作为产品或服务提供。

### 准备工作

- 安装`Elasticsearch`并配置运行
- 下载`Kibana`的压缩包

### 安装/配置/运行

1. 解压下载好的压缩包

2. 配置（配置文件位于安装目录（解压后的目录）的`config`目录中，文件名：`kibana.yml`）

   - 配置通配IP访问，修改`server.host`配置项

     ```yaml
     server.host: 0.0.0.0 # 默认值为 localhost
     ```

   - 配置`elasticsearch.url`

     ```yaml
     # 若和elasticsearch安装在一台虚拟机可以不用配置。默认为localhost
     elasticsearch.url: 192.168.136.101 
     ```

3. 运行

      ```shel
      # 进入安装目录
      bin/kibana
      ```

      - 查看运行结果，访问 `192.168.136.101:5601`

        ![](https://images.xushuai.fun/20190227214740.png)

      > 默认端口为：5601，可以通过 `server.port`配置项修改



> 到此`Elasticsearch`基本算是安装完毕



## 扩展：安装IK分词器

[Github：elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)

IK Analysis插件将Lucene IK分析器（<http://code.google.com/p/ik-analyzer/>）集成到elasticsearch中，支持自定义词典。

### 安装

- 方式1：下载IK分词器的压缩包，下载地址：[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)

  > 在 `elasticsearch`的安装目录中的`plugins`中创建`ik`目录
  >
  > 将下载的压缩包解压到创建的`ik`目录中

- 方式2：使用`elasticsearch-plugin`安装

  ```shell
  ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.6.1/elasticsearch-analysis-ik-6.6.1.zip
  ```

  **注意：版本号必须和你安装的版本一致，若此方法不行，请使用第一种方式。**

安装完毕，重启`elasticsearch`即可

