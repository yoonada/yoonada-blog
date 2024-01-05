---
title: Redis集群搭建
date: 2023-03-24 14:18:32
tags: [redis,CentOS7,集群]
categories: [redis,CentOS7,集群]
---
## Redis集群搭建
### 参考文章
https://cloud.tencent.com/developer/article/2124382?areaSource=&amp;traceId=
### 基础环境安装
* CentOS7服务器三台
  * 192.168.253.130
  * 192.168.253.131
  * 192.168.253.132
* 三台CentOS7服务器都安装redis-5.0.14
#### 基本安装（三台CentOS7都安装）
```shell
yum install gcc
```
```shell
cd /usr/local/
```
```shell
wget https://download.redis.io/releases/redis-5.0.14.tar.gz
```
```shell
tar -zxvf redis-5.0.14.tar.gz
```
```shell
cd redis-5.0.14
```
```shell
mkdir bin
```
```shell
make
```
```shell
cd src/
```
```shell
make install
```
```shell
cp mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server redis-sentinel /usr/local/redis-5.0.14/bin/
```
```shell
cd /usr/local/redis-5.0.14/bin
```
```shell
./redis-server ../redis.conf
```
### Redis的主从配置

下面给对应的CentOS7服务器进行配置，其中130为主机，131、132为从机

|     ip地址      | 端口号 |      角色      |
| :-------------: | :----: | :------------: |
| 192.168.253.130 |  6379  | 主机（master） |
| 192.168.253.131 |  6380  | 从机（slave）  |
| 192.168.253.132 |  6381  | 从机（slave）  |

#### 192.168.253.130的redis.conf修改

```shell
bind：0.0.0.0
port：6379
protected-mode：no
daemonize：yes
logfile：redis.log
requirepass：DD123456aa
masterauth：DD123456aa
```

#### 192.168.253.131的redis.conf修改

```shell
bind：0.0.0.0
port：6380
protected-mode：no
daemonize：yes
logfile：./redis.log
requirepass：DD123456aa
masterauth：DD123456aa
replicaof 192.168.253.130 6379 
```

#### 192.168.253.132的redis.conf修改

```shell
bind：0.0.0.0
port：6381
protected-mode：no
daemonize：yes
logfile：./redis.log
requirepass：DD123456aa
masterauth：DD123456aa
replicaof 192.168.253.130 6379 
```

### Redis的哨兵模式

#### 搭建（三台CentOS7对应的redis都进行如下操作）

```shell
cd /usr/local/redis-5.0.14
```
```shell
cp sentinel.conf sentinel.conf.bak
```
```shell
vim sentinel.conf
```
修改如下：
```shell
//端口默认为26379。
port:26379
//关闭保护模式，可以外部访问。
protected-mode:no
//设置为后台启动。
daemonize:yes
//日志文件。
logfile:sentinel.log
//指定主机IP地址和端口，并且指定当有2台哨兵认为主机挂了，则对主机进行容灾切换。
sentinel monitor mymaster 192.168.253.130 6379 2
//当在Redis实例中开启了requirepass，这里就需要提供密码。
sentinel auth-pass mymaster DD123456aa
//这里设置了主机多少秒无响应，则认为挂了。
sentinel down-after-milliseconds mymaster 3000
//主备切换时，最多有多少个slave同时对新的master进行同步，这里设置为默认的1。
sentinel parallel-syncs mymaster 1
//故障转移的超时时间，这里设置为三分钟。
sentinel failover-timeout mymaster 180000
```

```shell
cd /usr/local/redis-5.0.14/bin
```
```shell
./redis-sentinel ../sentinel.conf
```
```shell
ps -axu|grep redis
```

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230324152509371.png)

```shell
redis-cli -p 26379
```

```shell
info sentinel
```



![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230324153115011.png)
#### 容灾切换
现在我们模拟主机宕机，将主机 redis 服务关闭，如下

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230324153813734.png)

去原本的从机131、132看。会发现主机变为132了

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230324154028853.png)

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230324154357699.png)

### Redis集群模式
```shell
cd /usr/local/redis-5.0.14
```
```shell
mkdir cluster
```
```shell
cd cluster
```
```shell
mkdir 7000 7001 7002 7003 7004 7005
```
```shell
cp /usr/local/redis-5.0.14/redis.conf.bak /usr/local/redis-5.0.14/cluster/7000/redis.conf
```
每个文件夹里，要**对应一个redis.conf**（稍做修改）
```shell
# 端口号
bind  注释掉
port 7000
# 开启集群模式
cluster-enabled yes
# 设置保存节点配置文件的路径，默认值为 nodes-6379.conf。根据端口号更换
cluster-config-file  nodes-7000.conf
# 设置节点超时时间
cluster-node-timeout 5000
# 设置是否开启 aof 模式，对数据库完整性要求比较高可以开启
appendonly no
# 设置为后端启动
daemonize yes
protected-mode no
```
```shell
cp /usr/local/redis-5.0.14/src/redis-server /usr/local/redis-5.0.14/cluster/
```
```shell
cp /usr/local/redis-5.0.14/src/redis-cli /usr/local/redis-5.0.14/cluster/
```
```shell
vim start-all.sh
```
```shell
cd 7000
../redis-server ./redis.conf
cd ..
cd 7001
../redis-server ./redis.conf
cd ..
cd 7002
../redis-server ./redis.conf
cd ..
cd 7003
../redis-server ./redis.conf
cd ..
cd 7004
../redis-server ./redis.conf
cd ..
cd 7005
../redis-server ./redis.conf
cd ..
```

此时，拥有这些

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230327164305671.png)

执行脚本，启动这六个redis
```shell
sh start-all.sh
```

```shell
ps -aux | grep redis
```

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230327164343172.png)

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230327164422033.png)

启动集群

```shell
/usr/local/redis-5.0.14/cluster/redis-cli --cluster create 192.168.253.130:7000 192.168.253.130:7001 192.168.253.130:7002 192.168.253.130:7003 192.168.253.130:7004 192.168.253.130:7005 --cluster-replicas 1
```
