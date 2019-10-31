---
title: 配置Nginx反向代理
tags: 
  - ngxin
  - nginx配置反向代理
  - 反向代理
categories: nginx
description: Ngxin反向代理和负载均衡
photos:
  - >-
    https://www.imxushuai.com/img/asset/006ifTg0gy1fxq195rirbj314k0kat8k.jpg
abbrlink: 63282
date: 2018-07-27 18:06:11
---

<center><i>Ngxin反向代理和负载均衡</i></center>

![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxq195rirbj314k0kat8k.jpg)

<!-- more -->

<center><font size="6px">配置Nginx反向代理</font></center>


---
### 反向代理介绍

1. 反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。



2. 反向代理的作用：

> （1）保证内网的安全，可以使用反向代理提供WAF功能，阻止web攻击大型网站，通常将反向代理作为公网访问地址，Web服务器是内网。   
（2）负载均衡，通过反向代理服务器来优化网站的负载。

### 反向代理Demo
> 系统：centos 7   
此处使用tomcat作为被代理的服务器

1. 准备工作
```text
- centos 7安装jdk
- 上传tomcat到服务器（解压出两份，注意修改解压出来的目录名称）
- 安装nginx
```
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsanbicij3149091wft.jpg)

2. 修改tomcat配置，防止端口冲突（文件位置：`tomcat/conf/server.xml`）,只需要修改其中一个tomcat配置即可   
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsanr9wgj30mf02dt8t.jpg)
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsanml21j30my02daa8.jpg)
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsanioycj30n701zweo.jpg)

3. 修改ngxin配置(文件位置:`nginx/conf/nginx.conf`)
- 第一种配置方式
```conf
	upstream tomcat-test1 {
	    ## 设置被代理的ip
		server 192.168.200.132:8080;
    }
    server {
        listen       80;
        server_name  www.tomcat1.com;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            ## 被代理路径
            proxy_pass   http://192.168.200.132:8080;
            index  index.html index.htm;
        }
    }
	
	upstream tomcat-test2 {
		server 192.168.200.132:8081;
    }
    server {
        listen       80;
        server_name  www.tomcat2.com;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            proxy_pass   http://192.168.200.132:8081;
            index  index.html index.htm;
        }
    }
```
- 第二种配置方式
```conf
	upstream tomcat-test1 {
	    ## 设置被代理的ip
		server 192.168.200.132:8080;
    }
    server {
        listen       80;
        server_name  www.tomcat1.com;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            ## 被代理路径
            proxy_pass   http://tomcat-test1;
            index  index.html index.htm;
        }
    }
	
	upstream tomcat-test2 {
		server 192.168.200.132:8081;
    }
    server {
        listen       80;
        server_name  www.tomcat2.com;
 
        #charset koi8-r;
 
        #access_log  logs/host.access.log  main;
 
        location / {
            proxy_pass   http://tomcat-test2;
            index  index.html index.htm;
        }
    }
```

4. 运行tomcat，运行nginx
```shell
## tomcat 
tomcat-test1/bin/startup.sh
tomcat-test2/bin/startup.sh

## nginx
nginx/sbin/nginx
```

5. 测试
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsaog8yej30zc0b6gnx.jpg)
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmsao9ogij312409w40s.jpg)

### 配置均衡负载

1. 配置`nginx.conf`
```conf
    ## 轮询,每个节点一次
	upstream tomcat-test2 {
		server 192.168.200.132:8081;
		server 192.168.200.132:8082;
    }
    
    ## 设置权重
    upstream tomcat-test2 {
		server 192.168.200.132:8081;
		server 192.168.200.132:8082 weight=2; //权重越大，分配到的请求越多
    }

```

### 反向代理实例

1. 配置`nginx.conf`文件
```conf
    # erp项目
    upstream erp {
		server 192.168.183.130:8080;
    }
    server {
        listen       80;
        server_name  192.168.183.130:8080;
        ## 代理相关静态资源
        location / {
            proxy_pass   http://192.168.183.130:8080;
			proxy_read_timeout 600s;
			proxy_set_header  X-Real-IP  $remote_addr;
			proxy_set_header Host $host:$server_port;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         
            index  index.html index.htm;
        }
 
 
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

2. 测试
![](https://www.imxushuai.com/img/asset/006ifTg0gy1fxmt2zfz7nj31340hhq4e.jpg)
> 使用域名能访问到是因为我配置了hosts IP映射，如果没有配置，请使用IP访问
