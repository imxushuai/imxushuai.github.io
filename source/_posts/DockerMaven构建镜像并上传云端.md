---
title: DockerMaven构建镜像并上传云端
tags:
  - Docker
  - DockerMaven
  - DockerMaven部署
categories:
  - Docker
abbrlink: 51042
date: 2019-05-08 21:37:33
---

<center><i>使用 Docker-Maven-Plugin 完成 Jar 包镜像的构建并上传至云端</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/docker.jpg)

<!-- more -->

# 推送镜像到Docker Hub

## 配置maven setting文件

将`Docker hub`的账号密码配置在`Maven`的`setting.xml`文件中。将账号密码配置在`servers`节点中。

```xml
<servers>
    <server>
      <id>docker-hub</id>
      <username>imxushuai</username>
      <password>不告诉你</password>
      <configuration>
          <email>imxushuai@gmail.com</email>
      </configuration>
    </server>
</servers>
```

## 在项目中配置pom.xml

在项目中的`pom.xml`配置`MavenDocker`插件。

```xml
<build>
    <finalName>app</finalName>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.14</version>
            <configuration>
                <!-- 定义tag -->
                <imageName>imxushuai/${project.artifactId}:${project.version}</imageName>
                <baseImage>java</baseImage>
                <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
                <!-- 如果本地没有安装docker 可以使用远程docker  -->
                <!-- 不配置dockerHost，默认使用本地docker -->
                <dockerHost>http://192.168.136.104:2375</dockerHost>
                <serverId>docker-hub</serverId>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

- 注意：

  如果是`Spring Boot`项目，还需要配合`Spring Boot`的打包插件使用，将下方插件配置在pom.xml。

  ```xml
  <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
  ```

## 构建并推送镜像到Docker hub

在项目的命令行工具中执行命令。

我这里用的是`IDEA`自带的命令行：`Terminal`

```shell
maven clean package docker:build -DpushImage
```

**注意：上传至Docker hub，需要在本地安装Docker环境，或者在pom.xml中使用`dockerHost`路径连接远程服务器的Docker进行镜像的构建和上传。**

# 推送镜像到私服

## 配置宿主机Docker服务

要使用`DockerMaven`插件进行自动部署，需要让远程宿主机的`Docker`允许远程连接，修改`Docker`服务的`.service`文件。

1. 修改`docker.service`文件

   ```shell
   vim /lib/systemd/system/docker.service
   ```

   修改`ExecStart=/usr/bin/dockerd`在后面加上参数

   ```shell
   ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H fd:// --containerd=/run/containerd/containerd.sock
   ```

2. 重启Docker服务

   ```shell
   # 重新加载服务启动文件
   systemctl daemon-reload
   # 重启docker服务
   systemctl restart docker
   ```

3. 重启Docker仓库容器

   ```shell
   docker start imxushuai_registry
   ```

## 配置pom.xml

```xml
<build>
    <finalName>app</finalName>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.14</version>
            <configuration>
                <!-- 定义tag -->
                <imageName>192.168.136.104:5000/${project.artifactId}:${project.version}</imageName>
                <baseImage>jdk1.8</baseImage>
                <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
                <dockerHost>http://192.168.136.104:2375</dockerHost>
            </configuration>
        </plugin>
    </plugins>
</build>
```

- 注意：

  如果是`Spring Boot`项目，还需要配合`Spring Boot`的打包插件使用，将下方插件配置在pom.xml。

  ```xml
  <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
  ```

## 构建并推送镜像

执行上传命令

```shell
mvn clean package docker:build -DpushImage
```

OK！！！