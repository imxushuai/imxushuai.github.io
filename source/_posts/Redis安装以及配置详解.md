---
title: Redis安装以及配置详解
tags:
  - Redis
  - Redis安装
  - Redis配置详解
  - gcc-c++
categories:
  - Redis
abbrlink: 34757
date: 2018-05-20 20:12:49
---

<center><i>在CentOS下安装Redis以及Redis配置详解</i></center>

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/redis.jpg)

<!-- more -->

## 安装Redis依赖

`redis`是`C语言`开发的，安装`redis`需要先将官网上下载的源码进行编译，编译依赖`gcc`环境。

### 安装gcc-c++

有两种安装方式，一种是在线安装，一种是离线安装。

#### 在线安装

在线安装比较简单，使用`CentOS`自带的`yum`安装即可。

```shell
yum install gcc-c++ -y
```

#### 离线安装

离线安装就相对要复杂一点，需要安装一些`gcc`的一些依赖。下图是`gcc`环境需要的依赖。

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20180521192623602.png)

我这里已经全部下载好了，点击下面的百度网盘即可。

> 链接：[https://pan.baidu.com/s/1QDwYE1WC_vngJsqNSkZ5bQ](https://pan.baidu.com/s/1QDwYE1WC_vngJsqNSkZ5bQ)
>
> 提取码：ny2d

将下载好的这些`rpm`包上传到你的要安装的服务器，将这些`rpm`包放在同一个目录下。我这里全部放在了`/soft/新建文件夹`中。

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20180521193026633.png)

**安装**

```shell
rpm -Uvh *.rpm --nodeps --force
```

安装过程走完了过后，前往 /usr/bin 目录查看是否有 gcc和g++两个文件夹，如果有，说明安装成功。

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20180521193332441.png)

## 安装Redis

**提前将`Redis`的安装包上传至服务器。**

下载方式：

[官网下载](https://redis.io/download)

👇百度网盘下载👇

链接：[https://pan.baidu.com/s/1iSuc8MyOBsOJ3Mcy1nARWQ](https://pan.baidu.com/s/1iSuc8MyOBsOJ3Mcy1nARWQ)
提取码：25z1

### 解压Redis安装包

```shell
tar -zxvf redis-4.0.9.tar.gz
```

### 编译&&安装

进入解压后的Redis目录中。

```shell
# 进入redis目录
cd redis-4.0.9
# 编译&&安装  PREFIX指定安装目录
make && make install PREFIX=/usr/local/redis
```

### 启动测试

```shell
/usr/local/redis/bin/redis-server
```

![](https://dev.tencent.com/u/imxushuai/p/pic/git/raw/master/20190520182612.png)

## 扩展：配置以及开机启动

上面安装完毕后，其实已经可以直接使用。下面介绍下一些配置和开机启动。

### 配置

#### 配置项详解

| 配置项                          | 默认取值                                                     | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| daemonize                       | no                                                           | Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ） |
| pidfile                         | /var/run/redis.pid                                           | 当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定 |
| port                            | 6379                                                         | 指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字 |
| bind                            | 127.0.0.1                                                    | 绑定的主机地址                                               |
| timeout                         | 300                                                          | 当客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能 |
| loglevel                        | notice                                                       | 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice |
| logfile                         | stdout                                                       | 日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null |
| databases                       | 16                                                           | 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id |
| save <seconds> <changes>        | 默认配置文件中提供了三个条件：save 900 1/save 300 10/save 60 10000 | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
| rdbcompression                  | yes                                                          | 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| dbfilename                      | dump.rdb                                                     | 指定本地数据库文件名，默认值为 dump.rdb                      |
| dir                             | ./                                                           | 指定本地数据库存放目录                                       |
| slaveof <masterip> <masterport> | 无                                                           | 设置当本机为 slav 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步 |
| masterauth <master-password>    | 无                                                           | 当 master 服务设置了密码保护时，slav 服务连接 master 的密码  |
| requirepass                     | 无                                                           | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭 |
| maxclients                      | 无限                                                         | 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
| maxmemory <bytes>               |                                                              | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
| appendonly                      | no                                                           | 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no |
| appendfilename                  | appendonly.aof                                               | 指定更新日志文件名，默认为 appendonly.aof                    |
| appendfsync                     | everysec                                                     | 指定更新日志条件，共有 3 个可选值：no、always、everysec      |
| vm-enabled                      | no                                                           | 指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制） |
| vm-swap-file                    | /tmp/redis.swap                                              | 虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享 |
| vm-max-memory                   | 0                                                            | 将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0 |
| vm-page-size                    | 32                                                           | Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
| vm-pages                        | 134217728                                                    | 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
| vm-max-threads                  | 4                                                            | 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4 |
| glueoutputbuf                   | yes                                                          | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |
| activerehashing                 | yes                                                          | 指定是否激活重置哈希，默认为开启                             |
| include                         | /path/to/local.conf                                          | 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件 |
| hash-max-zipmap-entries         | 64                                                           | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |
| hash-max-zipmap-value           | 512                                                          | 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法 |

#### 获取与更新配置

##### 命令行获取配置

```shell
# 获取单个配置项
CONFIG GET ${CONFIG_SETTING_NAME}
# 获取全部配置,使用通配符 *
CONFIG GET *
```

使用具体配置项名称，取代`CONFIG_SETTING_NAME`。

##### 命令行更新配置

```shell
CONFIG SET ${CONFIG_SETTING_NAME} ${NEW_CONFIG_VALUE}
```

第一个参数为具体配置项，第二个参数该配置项的值。

##### 配置文件方式

1. 复制安装包下的`redis.conf`文件到`/usr/local/redis/bin`目录下

   ```shell
   cp /root/redis-4.0.9/redis.conf /usr/local/redis/bin
   ```

2. 通过编辑`/usr/local/redis/bin/redis.conf`来修改配置，在启动时，指定使用该配置文件启动即可。

   ```shell
   /usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
   ```

### 开机启动

#### 新建脚本文件

```shell
vim /etc/init.d/redis
```

> 如果没有安装`vim`，也可以使用`vi`

#### 文件内容

```shell
#!/bin/sh
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
PATH=/usr/local/bin:/sbin:/usr/bin:/bin

REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
REDIS_CLI=/usr/local/redis/bin/redis-cli

PIDFILE=/var/run/redis.pid

CONF="/usr/local/redis/bin/redis.conf"

case "$1" in  
    start)  
        if [ -f $PIDFILE ]  
        then  
                echo "$PIDFILE exists, process is already running or crashed"  
        else  
                echo "Starting Redis server..."  
                $EXEC $CONF  
        fi  
        if [ "$?"="0" ]   
        then  
              echo "Redis is running..."  
        fi  
        ;;  
    stop)  
        if [ ! -f $PIDFILE ]  
        then  
                echo "$PIDFILE does not exist, process is not running"  
        else  
                PID=$(cat $PIDFILE)  
                echo "Stopping ..."  
                $REDIS_CLI -p $REDISPORT SHUTDOWN  
                while [ -x ${PIDFILE} ]  
               do  
                    echo "Waiting for Redis to shutdown ..."  
                    sleep 1  
                done  
                echo "Redis stopped"  
        fi  
        ;;  
   restart|force-reload)  
        ${0} stop  
        ${0} start  
        ;;  
  *)  
    echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2  
        exit 1  
esac
```

**脚本部分内容解释:**

- `EXEC=/usr/local/redis/bin/redis-server`：执行脚本的地址

- `REDIS_CLI=/usr/local/redis/bin/redis-cli`：客户端执行脚本的地址

- `PIDFILE=/var/run/redis.pid`：进程id文件地址

- `CONF="/usr/local/redis/bin/redis.conf"`：配置文件地址

> 需要根据你的安装位置自行调整脚本中的参数。

#### 修改脚本文件权限

```shell
chmod 755 /etc/init.d/redis
```

#### 设置开机启动

```shell
chkconfig --add /etc/init.d/redis
chkconfig redis on
```
