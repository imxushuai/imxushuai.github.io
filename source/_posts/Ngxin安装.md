---
title: Ngxin安装
tags: 
  - nginx
  - nginx安装
  - 服务器代理
categories: nginx
description: 本文介绍在centos环境下安装nginx
photos:
  - >-
    https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxq195rirbj314k0kat8k.jpg
abbrlink: 7540
date: 2018-5-13 16:59:42
---

<center><i>CentOS环境下安装nginx</i></center>

![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxq195rirbj314k0kat8k.jpg)

<!-- more -->

<center><font size="6px">Nginx安装</font></center>

---
### Nginx介绍
Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。   

[Nginx官网](https://nginx.org/)

### 准备工作
> 系统：centos 7

1. 下载nginx压缩包并上传至centos系统
2. 安装nginx需要的第三方库
```shell
## PCRE：PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。
yum install -y pcre pcre-devel

## zlib：zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。
yum install -y zlib zlib-devel

## openssl：OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。
yum install -y openssl openssl-devel
```

### 安装

1. 解压nginx压缩包
```shell
## 该压缩包路径为当前目录
tar -zxvf nginx-1.8.0.tar.gz
```

2. 使用nginx主目录中的configure命令生成makefile文件（将下面的命令一次性全部复制并执行）
```shell
## 进入nginx目录(版本号以你下载的版本号为准，请注意修改)
cd nginx-1.8.0

## 创建nginx临时目录
mkdir -p /var/temp/nginx

## 复制下方全部，执行
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_stub_status_module --with-http_ssl_module
```

3. 执行生成的Makefile文件   
![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxmray6fy8j30q402nglr.jpg)
```shell
## 编译Makefile文件
make

## 执行安装
make install
```

4. 进入`/usr/local/`查看是否安装成功
```shell
cd /usr/local
```
![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxmrbpo0pmj30um02gdfw.jpg)

5. 启动 nginx
```shell
## 启动nginx（当前目录为：/usr/local）
nginx/sbin/nginx
```

6. 查看是否成功启动
```shell
ps aux|grep nginx
```
![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxmrbpwbgcj311d034dg4.jpg)

7. 访问nginx服务器
![](https://raw.githubusercontent.com/imxushuai/ForPicGo/master/006ifTg0gy1fxmrbq1202j30wo0dqaas.jpg)

> 其他命令   
 
- 停止命令:  

```shell
## 两个命令都可以完成停止操作
./nginx -s stop   
./nginx -s quit   //建议使用这个命令
```

- 重启

```shell
./nginx -s reload
```
