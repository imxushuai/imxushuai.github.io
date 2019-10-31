---
title: Rancher入门
tags:
  - Rancher
  - Rancher安装
  - 容器管理
categories:
  - Rancher
abbrlink: 34463
date: 2019-05-16 22:56:45
---

<center><i>容器管理工具 Rancher 的安装和使用</i></center>

![](https://www.imxushuai.com/img/asset/rancher.png)

<!-- more -->

# Rancher介绍

Rancher官方文档：[Rancher](<https://rancher.com/docs/rancher/v1.6/zh/>)

## 什么是Rancher

Rancher是一个开源的企业级容器管理平台。通过Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台。

## Rancher特性

### 基础设置编排

Rancher可以使用任何公有云或者私有云的Linux主机资源。Linux主机可以是虚拟机，也可以是物理机。Rancher仅需要主机有CPU，内存，本地磁盘和网络资源。从Rancher的角度来说，一台云厂商提供的云主机和一台自己的物理机是一样的。

Rancher为运行容器化的应用实现了一层灵活的基础设施服务。Rancher的基础设施服务包括网络， 存储， 负载均衡， DNS和安全模块。Rancher的基础设施服务也是通过容器部署的，所以同样Rancher的基础设施服务可以运行在任何Linux主机上。

### 容器编排与调度

很多用户都会选择使用容器编排调度框架来运行容器化应用。Rancher包含了当前全部主流的编排调度引擎，例如[Docker Swarm](https://rancher.com/docs/rancher/v1.6/zh/swarm)， [Kubernetes](https://rancher.com/docs/rancher/v1.6/zh/kubernetes)， 和[Mesos](https://rancher.com/docs/rancher/v1.6/zh/mesos/)。同一个用户可以创建Swarm或者Kubernetes集群。并且可以使用原生的Swarm或者Kubernetes工具管理应用。

除了Swarm，Kubernetes和Mesos之外，Rancher还支持自己的Cattle容器编排调度引擎。Cattle被广泛用于编排Rancher自己的基础设施服务以及用于Swarm集群，Kubernetes集群和Mesos集群的配置，管理与升级。

### 应用商店

Rancher的用户可以在[应用商店](https://rancher.com/docs/rancher/v1.6/zh/catalog)里一键部署由多个容器组成的应用。用户可以管理这个部署的应用，并且可以在这个应用有新的可用版本时进行自动化的升级。Rancher提供了一个由Rancher社区维护的应用商店，其中包括了一系列的流行应用。Rancher的用户也可以[创建自己的私有应用商店](https://rancher.com/docs/rancher/v1.6/zh/catalog/private-catalog/)。

### 企业级权限控制

Rancher支持灵活的插件式的用户认证。支持Active Directory，LDAP， Github等 [认证方式](https://rancher.com/docs/rancher/v1.6/zh/configuration/access-control/)。 Rancher支持在[环境](https://rancher.com/docs/rancher/v1.6/zh/environments/)级别的基于角色的访问控制 (RBAC)，可以通过角色来配置某个用户或者用户组对开发环境或者生产环境的访问权限。

下图展示了Rancher的主要组件和功能：

![](https://www.imxushuai.com/img/asset/rancher_overview_2.png)

> 综上所述，反正吹爆就完事了。



# Rancher安装

## Docker 安装

1. 拉取Rancher镜像

   ```shell
   docker pull rancher/server
   ```

2. 制作容器并运行

   ```shell
   docker run -di --name=rancher --restart=unless-stopped -p 8080:8080 rancher/server
   ```

3. 访问`http://ip:port`

   ![](https://www.imxushuai.com/img/asset/20190701152621.png)

   安装完毕。

# Rancher使用

## 添加应用

点击`用户应用 -> 添加应用`，输入`名称`和`描述`，最后点击`创建`即可。

![](https://www.imxushuai.com/img/asset/20190701152748.png)

## 添加主机

这里有个地方注意：如果你的机器是你自建的服务器，比如说：虚拟机。

再第一步中，选择`驱动`时，就选择`Custome`。

1. 进入`基础架构 -> 主机 -> 添加主机`，然后按下图操作：

   ![](https://www.imxushuai.com/img/asset/20190701154928.png)

   **注意：需要开放UDP的`500`和`4500`端口，需要填写IP后，再复制命令到服务器执行，命令是输入IP时同步生成的。执行了命令后可直接点击关闭，关闭添加主机页面**

2. 等待服务器执行命令完成后，刷新`主机列表`，就会显示添加好的主机

   ![](https://www.imxushuai.com/img/asset/20190701160935.png)

3. 查看主机情况

   ![](https://www.imxushuai.com/img/asset/20190701180240.png)

## 添加服务

进入`应用 -> 全部`，选择第一步中创建的应用，选择`添加服务`

- ![](https://www.imxushuai.com/img/asset/20190701164101.png)

- 录入基本信息

  ![](https://www.imxushuai.com/img/asset/20190701161508.png)

  选择镜像，建议到服务器粘贴复制，免得手写输错。

- 其他信息，例如：挂载目录

  ![](https://www.imxushuai.com/img/asset/20190701161508.png)

  我这里创建的是`Mongodb`的容器，所以不需要特别的配置，只需要录入基本信息就OK了。

- 点击`创建`

- 远程连接一下`mongodb`容器，测试是否能连接上

  ![](https://www.imxushuai.com/img/asset/20190701180723.png)

  成功连接！





