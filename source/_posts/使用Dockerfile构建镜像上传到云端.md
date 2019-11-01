---
title: 使用Dockerfile构建镜像上传到云端
tags:
  - docker
  - docker镜像上传
  - docker hub
  - docker私有仓库
categories:
  - Docker
abbrlink: 60185
date: 2019-05-06 23:46:34
---

<center><i>Dockerfile构建镜像分别上传到 Docker hub和私有仓库</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/docker.jpg)

<!-- more -->

# Dockerfile

Dockerfile是用于构建Docker镜像的脚本文件。

## 常用命令

| 命令                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
| MAINTAINER user_name               | 声明镜像的创建者                                             |
| ENV key value                      | 设置环境变量 (可以写多条)                                    |
| RUN command                        | 是Dockerfile的核心部分(可以写多条)                           |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
| WORKDIR path_dir                   | 设置工作目录                                                 |
| EXPOSE port1 prot2                 | 用来指定端口，使容器内的应用可以通过端口和外界交互           |
| CMD argument                       | 在构建容器时使用，会被docker run 后的argument覆盖            |
| ENTRYPOINT argument                | 和CMD相似，但是并不会被docker run指定的参数覆盖              |
| VOLUME                             | 将本地文件夹或者其他容器的文件挂载到容器中                   |

## 构建JDK镜像

***编写 Dockerfile 构建 JDK 镜像***

1. 上传`JDK`到宿主机

2. 创建Dockerfile

   ```shell
   vim Dockerfile
   ```

   文件内容如下：

   ```shell
   FROM centos:7
   MAINTAINER imxushuai
   # 切换工作目录
   WORKDIR /usr
   # 执行命令
   RUN mkdir -p /usr/local/java
   # 从宿主机复制文件到镜像
   ADD jdk-8u201-linux-x64.tar.gz /usr/local/java
   
   # 配置环境变量
   ENV JAVA_HOME /usr/local/java/jdk1.8.0_201
   ENV JRE_HOME $JAVA_HOME/jre
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
   ENV PATH $JAVA_HOME/bin:$PATH
   ```

3. 构建镜像

   ```shell
   docker build -t='jdk1.8' .
   ```

4. 查看镜像是否成功生成

   ```shell
   docker images
   ```

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626220302.png)

5. 创建容器

   ```shell
   docker run -rm -it --name=myjdk jdk1.8 /bin/bash
   ```

   查看Java版本

   ```shell
   java -version
   ```

   JDK构建成功

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626220834.png)



# 上传镜像到云

将自己生成的镜像上传到云，云端可以是自己的私有仓库，也可以是类似`Docker hub`的共有仓库。

## 上传到Docker Hub

### 创建Docker hub账号（省略）

emmmmm，正常创建即可，实在不会就百度/谷歌一下。

### 上传镜像

1. Docker hub创建仓库

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626222248.png)

2. 宿主机登录到Docker Hub

   ```shell
   docker login
   ```

   输入账号密码即可完成登录！

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626221946.png)

3. 上传镜像到`Docker hub`

   ```shell
   # 将生成的镜像打上tag，需要在镜像名称加上Docker hub用户名
   # tag名称还需要和Docker hub一致
   # 如我上面创建的仓库为：imxushuai/jdk1.8
   # 所以下面我的tag为：imxushuai/jdk1.8
   docker tag jdk1.8 imxushuai/jdk1.8
   # 上传
   docker push imxushuai/jdk1.8
   ```

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626223638.png)

   由于Docker hub在国外，所以上传可能会花费一些时间，请耐心等待！！！（如果有科学上网的办法，那就真是太棒了）

4. Docker hub查看是否上传成功

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626223721.png)

   这样就可以在其他能联网的电脑下载到上传到Docker hub的镜像。

   拉取镜像：`docker pull imxushuai/jdk1.8`

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626225046.png)

## 上传到私有仓库

将镜像上传到由自己搭建的私有仓库

### 搭建私有仓库

1. 拉取私有仓库镜像

   ```shell
   docker pull registry
   ```

2. 运行私有仓库容器

   ```shell
   docker run -di --name=imxushuai_registry -p 5000:5000 registry
   ```

   访问测试：http://192.168.136.104:5000/v2/_catalog

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626231324.png)

3. 修改宿主机`daemon.json`

   ```shell
   vim /etc/docker/daemon.json
   ```

   文件内容如下：

   ```shell
   {"insecure-registries":["192.168.136.104:5000"]}
   ```

4. 重启宿主机`docker`

   ```shell
   systemctl restart docker
   ```

   注意：重启`docker`后，需要手动重启容器哦！

### 上传镜像

1. 给需要上传到私有仓库的镜像打上tag

   ```shell
   # 将生成的镜像打上tag，需要在镜像名称加上Docker hub用户名
   # tag名称还需要和Docker hub一致
   # 如我上面创建的仓库为：imxushuai/jdk1.8
   # 所以下面我的tag为：imxushuai/jdk1.8
   docker tag jdk1.8 192.168.136.104:5000/jdk1.8
   ```

2. 上传镜像

   ```shell
   docker push 192.168.136.104:5000/jdk1.8
   ```

3. 查看是否上传成功

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190626233036.png)

   OK！！！