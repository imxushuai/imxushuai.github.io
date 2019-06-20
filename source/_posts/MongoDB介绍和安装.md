---
title: MongoDB介绍和安装
tags:
  - MongoDB
  - MongoDB教程
  - MongoDB安装
categories:
  - MongoDB
date: 2019-04-20 16:46:36
---

<center><i>MongoDB简介以及安装</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/mongoDB.png)

<!-- more -->

# MongoDB介绍

## MongoDB简介

`MongoDB` 是由`C++`语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

`MongoDB` 旨在为WEB应用提供可扩展的高性能数据存储解决方案。

`MongoDB` 将数据存储为一个文档，数据结构由键值(key=>value)对组成。`MongoDB `文档类似于 `JSON` 对象。字段值可以包含其他文档，数组及文档数组。

## MongoDB特点

`MongoDB` 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。它是一个面向集合的,模式自由的文档型数据库。
具体特点总结如下：

- 面向集合存储，易于存储对象类型的数据
- 模式自由
- 支持动态查询
- 支持完全索引，包含内部对象
- 支持复制和故障恢复
- 使用高效的二进制数据存储，包括大型对象（如视频等）
- 自动处理碎片，以支持云计算层次的扩展性
- 支持Python，PHP，Ruby，Java，C，C#，Javascript，Perl 及C++语言的驱动程
  序，社区中也提供了对Erlang 及.NET 等平台的驱动程序
- 文件存储格式为`BSON`（一种`JSON`的扩展）

## MongoDB基本概念

MongoDB 的逻辑结构是一种层次结构。主要构成分别为：

- 文档(document)：相当于关系数据库中的一行记录
- 集合(collection)：相当于关系数据库的表
- 数据库(database)：逻辑上组织在一起，就是数据库（database）

**一个`MongoDB` 实例支持多个数据库（database）。`文档(document)`、`集合(collection)`、`数据库(database)`的层次结构。**

### 扩展

与关系型数据库对应关系表格，截取自[[菜鸟教程](https://www.runoob.com/mongodb/mongodb-databases-documents-collections.html)]：

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| :----------- | :--------------- | :---------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |

看下图更直观：

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/Figure-1-Mapping-Table-to-Collection-1.png)

# MongoDB安装

**注意：下面的安装环境为：`CentOS 7`**

## .tgz安装

官方下载地址：[download-center](https://www.mongodb.com/download-center/community)

1. 选择`MongoDB`版本和操作系统版本，我这里是安装到`CentOS 7`上。

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616212311.png)

2. 下载完毕后将下载的`.tgz`的包上传到服务器。

   ![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560692109626.png)

3. 解压压缩包

   ```shell
   tar -zxvf mongodb-linux-x86_64-4.0.10.tgz
   ```

4. 创建数据存放目录

   `MongoDB`的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。

   ```shell
   mkdir -p /date/db
   ```

   也可以在启动时，使用`--dbpath`手动指定数据存放位置。

5. 运行

   进入解压后的文件夹中的`bin`目录，执行下方命令

   ```shell
   ./mongod
   ```

6. 查看是否成功启动

   ```shell
   # 如果没有此命令，需要安装net-tools：yum install net-tools -y
   netstat -neplt
   ```

   ![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560692613635.png)

7. 绑定通配IP访问

   在上面安装完成后，可以看到`MongoDB`的`27017`端口监听的IP是`127.0.0.1`，这就意味着，外网是连不到的，这显然不是我们想要的效果，这里需要配置外网IP也能够访问到安装好的`MongoDB`。

   ```shell
   # 停止之前的MongoDB
   kill -9 19353
   # 使用参数启动并绑定通配IP访问，更多参数可以使用 ./mongod --help查看（全英文）
   ./mongod --bind_ip_all
   # 再提一嘴，这里都是前台运行，意思就是关掉当前连接的session，mongodb就没了
   # 我们可以使用 --fork 和 --logpath 配合使用（如果要使用--fork就必须使用--logpath）
   # 日志路径随意，你喜欢就行，但是如果日志目录不存在需要先行创建
   ./mongod --bnd_ip_all --fork --logpath=/logs/mongodb/mongodb.log
   ```

8. 再次查看

   ```shell
   netstat -neplt
   ```

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616220302.png)

   搞定！！！

## yum安装

> 注意：yum安装方式必须在服务器有网的情况下才行，否则请使用`.tgz`方式安装

1. 创建`MongoDB`的`yum`源文件

   ```shell
   vim /etc/yum.repos.d/mongodb-org-4.0.repo
   ```

   文件内容如下：

   ```tex
   [mongodb-org-4.0]
   name=MongoDB Repository
   baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
   gpgcheck=1
   enabled=1
   gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
   ```

2. 安装`MongoDB`

   ```shell
   # 安装可能会花费一些时间，请耐心等待
   yum install -y mongodb-org
   ```

   这样是安装的最新版本，如果需要指定的版本话可以执行下面这条命令并修改版本号

   ```shell
   yum install -y mongodb-org-4.0.10 mongodb-org-server-4.0.10 mongodb-org-shell-4.0.10 mongodb-org-mongos-4.0.10 mongodb-org-tools-4.0.10
   ```

3. 创建`MongoDB`需要的文件夹并设置权限

   ```shell
   # 创建目录
   mkdir -p /var/lib/mongo
   mkdir -p /var/log/mongodb
   # 设置权限
   chown -R mongod:mongod /var/lib/mongo
   chown -R mongod:mongod /var/log/mongodb
   ```

4. 启动

   ```shell
   systemctl start mongod
   ```

5. 查看是否成功启动

   ```shell
   netstat -neplt
   ```

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616231210.png)

6. 配置通配IP访问

   ```shell
   # 修改配置文件（yum安装才自带配置文件哦）
   # .tgz方式安装需要配置文件的话需要自己创建，启动时使用 -f 指定配置文件位置
   vim /etc/mongod.conf
   ```

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616231600.png)

7. 重新启动

   ```shell
   systemctl restart mongod.service
   ```

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616231736.png)

   搞定！！！

## 使用Docker安装

> 注意：需要先安装`docker`环境。参考：[docker安装](<https://www.runoob.com/docker/centos-docker-install.html>)

1. 搜索`MongoDB`镜像

   ```shell
   docker search mongodb
   ```

   ![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/20190616232951.png)

2. 拉取`MongoDB`镜像

   ```shell
   docker pull mongo
   ```

3. 使用镜像创建容器

   ```shell
   docker run -di --name=mymongodb -p 27017:27017 mongo
   ```

> hey，is done，f**k easy？？！！

