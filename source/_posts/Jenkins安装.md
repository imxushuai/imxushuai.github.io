---
title: Jenkins安装
tags:
  - Jenkins
  - Jenkins安装
  - Docker安装Jenkins
  - 持续集成
categories:
  - 持续集成
  - Jenkins
abbrlink: 4921
date: 2019-05-13 18:37:44
---

<center><i>手把手教你如何安装 Jenkins</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/Jenkins.png)

<!-- more -->

# Jenkins介绍

Jenkins官网：[Jenkins](<https://jenkins.io/zh/>)

## 什么是Jenkins

Jenkins是开源CI&CD软件领导者， 提供超过1000个插件来支持构建、部署、自动化， 满足任何项目的需要。

Jenkins 支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序。

PS：关于自动化部署，我的博客使用的是`Hexo` + `GitHub Pages` + `Travis CI`

## Jenkins特性

- 持续集成和持续交付

- 简易安装/建议配置

- 可扩展插件

- 支持分布式

  ......



# Jenkins安装

***建议使用 RPM 包安装方式***

## RPM包安装

请提前安装`Java`环境，即安装`JDK`，`CentOS`，以下操作默认已经安装好`JDK`并配置好了环境变量。

1. 下载`RPM`包

   官方下载地址：[https://jenkins.io/zh/download/](<https://jenkins.io/zh/download/>)

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190628234509.png)

   我这里的安装环境是`CentOS 7`，所以我选择的是：`Red Hat/Fedora/CentOS`（建议选择长期支持版本）

2. 上传`RPM`到服务器

3. 安装`RPM`包

   ```shell
   # 安装 -ivh 更新使用 -Uvh
   rpm -ivh jenkins-2.176.1-1.1.noarch.rpm
   ```

4. 修改配置文件

   ```shell
   vim /etc/sysconfig/jenkins
   ```

   分别修改：`JENKINS_USER`和`JENKINS_PORT`

   ```shell
   # 直接使用root用户，或者自行创建jenkins用户
   JENKINS_USER="root"
   # 默认是8080端口，建议修改成别的端口
   JENKINS_PORT="18080"
   ```

5. 启动`Jenkins`

   ```shell
   systemctl start jenkins
   ```

## Yum安装

1. 安装自动索引`yum`源的包。（可选操作）

   ```shell
   yum install yum-fastestmirror -y
   ```

2. 添加`Jenkins`的`yum`源

   ```shell
   wget -O /etc/yum.repos.d/jenkins.repo http://jenkins-ci.org/redhat/jenkins.repo
   ```

3. 获取`Jenkins`的包签名密钥

   ```shell
   rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
   ```

4. 安装`Jenkins`

   ```shell
   yum install jenkins -y
   ```

5. 启动

   ```shell
   # 若需要配置，配置文件位于：/etc/sysconfig/jenkins
   service jenkins start
   ```

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630200749.png)

6. 访问`http://ip:port/8080`

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190630200939.png)

## Docker安装

1. 拉取`Jenkins`镜像

   ```shell
   docker pull jenkins
   ```

2. 创建本地挂载目录

   ```shell
   # 创建目录
   mkdir -p /var/jenkins
   # 修改目录权限,不然挂载的时候会报错
   chown -R 1000:1000 /var/jenkins
   ```

3. 使用镜像运行容器

   ```shell
   docker run -di --name=jenkins -p 18080:8080 -p 5000:5000 --privileged=true -v /var/jenkins:/var/jenkins_home jenkins
   ```

   我这里将容器的`8080`端口挂载到了宿主机的`18080`端口，因为`8080`这个端口太敏感了，最后将容器内的`/var/jenkins_home`目录挂载到了宿主机的`/var/jenkins`目录。

4. 访问测试

   浏览器访问：`http://47.107.128.204:18080`

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190629002927.png)

Docker Jenkins安装完毕

# Jenkins初始化

安装完毕后，访问`Jenkins`需要做一些初始化配置。

1. 获取管理员密码

   - 方式1：查看挂载到本地目录的密码文件

     ```shell
     cat /var/jenkins/secrets/initialAdminPassword
     ```

     ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190629003259.png)

   - 方式2：查看容器内的密码文件

     ```shell
     docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
     ```

   建议直接使用第一种方式，没有必要使用第二种，除非你没有挂载目录到本地。

2. 将获取到的密码输入到密码框中，点击`Continue`，然后选择安装插件，选择左侧安装默认插件即可。

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190629003528.png)

   紧接着我们就进入了漫长的安装过程，emmmmmm......

3. 最后新建并保存用户，就可以开始使用`Jenkins`啦