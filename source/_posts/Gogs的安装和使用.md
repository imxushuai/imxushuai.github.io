---
title: Gogs的安装和使用
tags:
  - Gogs
  - Gogs安装
  - Docker安装Gogs
  - Git服务器
categories:
  - Gogs
abbrlink: 19914
date: 2019-05-12 15:32:50
---

<center><i>使用 Gogs 自建 Git 服务器</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/Gogs.png)

<!-- more -->

# Gogs介绍

## 什么是Gogs

Gogs 是一款极易搭建的自助 Git 服务。

Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 **所有平台**，包括 Linux、Mac OS X、Windows 以及 ARM 平台。

可以理解为简易的`GitHub`。

##  Gogs功能特性

- 支持活动时间线
- 支持`SSH`以及`HTTP/HTTPS`协议
- 支持`SMTP`、`LDAP`和反向代理的用户认证
- 支持反向代理子路径
- 支持用户、组织和仓库管理系统
- 支持仓库和组织界别Web钩子（包括Slack集成）
- 支持仓库`Git`钩子和部署密钥
- 支持仓库工单（Issue）、合并请求（Pull Request）和WiKi
- 支持添加和删除仓库协作者
- 支持`Gravatar`以及自定义源
- 支持邮件服务
- 支持后台管理面板
- 支持Mysql、PostgreSQL、SQLite3和TiDB数据库
- 支持多语言

# Gogs安装

## 二进制安装

1. 下载二进制包

   下载地址：[gogs下载](<https://gogs.io/docs/installation/install_from_binary>)

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630113004.png)

2. 将下载的包上传至服务器

3. 解压压缩包并进入解压后的安装目录

   ```shell
   # 解压安装包
   tar -zxvf gogs_0.11.86_linux_amd64.tar.gz
   # 进入安装目录
   cd gogs
   ```

4. 运行`Gogs`

   ```shell
   # 前台运行
   ./gogs web
   ```

   
   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630113321.png)
   
   成功运行！
   
   后台运行：`nohup ./gogs web`

## Docker安装Gogs

1. 拉取`Gogs`镜像

   ```shell
   docker pull gogs/gogs
    ```

2. 运行容器

   ```shell
   docker run -di --name=gogs -p 22222:22 -p 3000:3000 -v /var/gogsdata:/data gogs/gogs
   ```

   OK！！就启动完成了。

# Gogs页面初始化

1. 初始化配置，只需要修改部分配置

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190628223713.png)

   - 数据库类型：测试使用`SQLite3`即可，实际可以选择使用`Mysql`或`PostgreSQL`
   - 域名：使用安装`Gogs`的服务器的IP
   - 应用URL：使用安装`Gogs`的服务器`URL`或者域名
   - 运行系统用户：具体看你使用的是什么用户，我使用的`root`，所以输入`root`

   然后点击`立即安装`即可。

2. 注册账号，然后就可以创建仓库开始使用了。

# Gogs的基本使用

## 创建仓库

1. 登录账号

2. 点击新建仓库

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630143510.png)

3. 填写仓库信息后，点击`创建仓库`。

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630143705.png)

   创建仓库完毕，和Github区别不是太大。