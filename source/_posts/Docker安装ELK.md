---
title: Docker安装ELK
tags:
  - ELK
  - Elasticsearch
  - Logstash
  - kibana
  - Docker安装Elasticsearch
  - Docker安装Logstash
  - Docker安装Kibana
categories:
  - 全文检索
  - elasticsearch
abbrlink: 65404
date: 2019-05-01 16:51:31
---

<center><i>使用 Docker 安装 E(Elasticearch) L(Logstash) K(Kibana)</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/ELK.png)

<!-- more -->

# 所需环境

1. `CentOS 7`服务器一台（云主机或虚拟机均可）
2. 在`CentOS 7`提前安装`Docker`环境且能够连接网络（下载镜像需要网络）

# 安装Elasticsearch

1. 搜索`Elasticsearch`镜像

   ```shell
   docker search elasticsearch
   ```

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619213440.png)

   > 不建议使用最新版本，我这里下载6.6.1版本，和我另一篇文章中使用的版本一致。

2. 拉取镜像

   ```shell
   docker pull elasticsearch:6.6.1
   ```

   **国内下载镜像速度有点感人，耐心等待。。**

3. 创建用户定义的网络（用于连接到同一网络的其他服务，例如Kibana）：

   ```shell
   docker network create somenetwork
   ```

4. 创建容器

   需要在Docker宿主机创建`elasticsearch`配置文件挂载到容器中，方便修改配置。

   ```shell
   vim /etc/elasticsearch.yml
   ```

   文件内容：

   ```yaml
   cluster.name: "docker-cluster"
   network.host: 0.0.0.0
   # 允许任何端口访问
   transport.host: 0.0.0.0
   ```

   启动创建容器

   ```shell
   docker run -di --name=myelasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /etc/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:6.6.1
   ```

5. 查看是否成功运行

   ```shell
   docker ps
   ```

   成功运行如下：

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619215833.png)

6. 页面访问elasticsearch

   如果得到下面类似返回，说明启动成功。

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619221154.png)

8. 以后修改配置文件后，重启`docker`容器即可。

   ```shell
   # 重启elasticsearch容器
   docker restart myelasticsearch
   ```

8. 安装ik分词器

   ```shell
   # 进入容器
   docker exec -it myelasticsearch /bin/bash
   # 运行安装命令
   /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.6.1/elasticsearch-analysis-ik-6.6.1.zip
   ```

   安装完毕后重启`elasticsearch`容器即可。

9. 测试ik分词器

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619235926.png)

# 安装Logstash

1. 拉取`logstash`镜像

   注意：这里需要从elasitc官方镜像库拉取镜像哦，docker hub上的镜像我试的时候是不可用的，不知道是什么原因。

   ```shell
   docker pull docker.elastic.co/logstash/logstash:6.6.2
   # 如果拉取速度过慢，可以拉取我传到docker hub上的镜像
   # 与官方6.6.2的镜像是一致的
   # docker pull docker.io/imxushuai/logstash:6.6.2
   ```

2. 创建容器

   ```shell
   docker run --rm -di docker.elastic.co/logstash/logstash:6.6.2
   ```

3. 查看容器ID

   ```shell
   docker ps
   ```

4. 将容器中的配置文件复制到宿主机

   ```shell
   # 创建用于存放配置的目录
   mkdir -p /etc/logstash
   # 复制配置文件，冒号前面为容器ID
   docker cp d7405af81c00:/usr/share/logstash/config /etc/logstash/config/
   # 复制logstash管道文件
   docker cp d7405af81c00:/usr/share/logstash/pipeline /etc/logstash/pipeline
   ```

5. 重新创建容器，并挂载上面两个文件夹

   ```shell
   # 停止原有容器，否则无法删除
   docker stop d7405af81c00
   # 删除原有容器，使用容器ID删除
   docker rm d7405af81c00
   # 创建新容器并挂载目录
   docker run -di --name=mylogstash -v /etc/logstash/config:/usr/share/logstash/config -v /etc/logstash/pipeline:/usr/share/logstash/pipeline docker.elastic.co/logstash/logstash:6.6.2
   ```

6. 使用时只需要将`/etc/logstash/pipeline/logstash.conf`替换为要执行的输入输出，重启容器即可。

7. 日志查看（排错必备）

   ```shell
   # logstash运行也需要一些时间，日志会持续打印
   docker logs -f --tail=30 mylogstash
   ```

8. 插件安装需要进入容器内部

   ```shell
   # 进入容器
   docker exec -it mylogstash bash
   # 进入logstash的bin目录
   cd /usr/share/logstash/bin
   # 安装插件,我这里的示例是安装的jdbc
   ./logstash-plugin install logstash-input-jdbc
   ```

# 安装Kibana

1. 拉取镜像

   ```shell
   docker pull kibana:6.6.1
   ```

2. 启动容器

   ```shell
   docker run -di -p 5601:5601 kibana:6.6.1
   ```

3. 查看容器ID

   ```shell
   docker ps
   ```

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619232531.png)

4. 将容器中的配置文件复制到宿主机

   ```shell
   # 创建存放配置文件的目录
   mkdir -p /etc/kibana
   # 复制
   docker cp 3286d9b5d6b4:/usr/share/kibana/config /etc/kibana/config
   ```

5. 重新创建容器，并挂载配置文件目录

   ```shell
   # 停止原有容器
   docker stop 3286d9b5d6b4
   # 删除原有容器
   docker rm 3286d9b5d6b4
   # 创建新容器
   docker run -di --name=mykibana --net somenetwork -p 5601:5601 -v /etc/kibana/config:/usr/share/kibana/config kibana:6.6.1
   ```

6. 修改配置文件

   ```shell
   # 修改kibana配置文件
   vim /etc/kibana/config/kibana.yml
   ```

   修改后的配置文件内容如下：

   ```yaml
   server.name: kibana
   # 允许所有地址访问
   server.host: "0.0.0.0"
   # elasticsearch的地址
   elasticsearch.url: http://192.168.136.104:9200
   xpack.monitoring.ui.container.elasticsearch.enabled: true
   ```

7. 重启kibana

   ```shell
   docker restart mykibana
   ```

8. 访问测试

   我事先在`elasticsearch`中新增了名为`test`的索引库，成功查询到`test`索引库。

   ![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190619234107.png)

