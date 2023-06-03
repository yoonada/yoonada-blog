---
title: docker搭建MySQL的主从复制
date: 2022-10-31 17:06:23
tags: [centos,docker,MySQL]
categories: [centos,docker,MySQL]
---
### docker拉取MySQL5.7的镜像
```shell
docker pull mysql:5.7
```
### 创建主库的数据目录
```shell
mkdir -p /usr/local/mysql/mysql-master/{data,conf,log}
```
### 创建主库配置文件my.cnf
```shell
vim /usr/local/mysql/mysql-master/conf/my.cnf
```
```shell
[mysqld]
## 设置server_id，在同一局域网中需要唯一
server_id=1
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 指定需要同步的数据库
## binlog-do-db=db1
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 数据库时区
default-time_zone='+8:00'
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（row-行级别；statement=语句级别；mixed-混合级别）
binlog_format=mixed
## 二进制日志过期清理时间。默认为0，表示不自动清理
expire_logs_days=7
## 忽略主从复制中遇到的所有错误活指定的错误类型，避免slave端复制中断
## 如：1062错误码代表主键重复；1032错误码代表主从数据库数据不一致
slave_skip_errors=1062
```
### 启动主库
```shell
docker run -d -p 3308:3306 --privileged=true \
-v /usr/local/mysql/mysql-master/log:/var/log/mysql  \
-v /usr/local/mysql/mysql-master/data:/var/lib/mysql  \
-v /usr/local/mysql/mysql-master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root123456 \
--name mysql-master mysql:5.7
```
### 查看主库容器的ip并记录起来
```shell
docker inspect mysql-master
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202211011443887.png)
### 进入主库容器
```shell
docker exec -it mysql-master /bin/bash
```
### 登录主库
```shell
mysql -uroot -proot123456 -P3308
```
### 创建slave数据同步用户
```shell
CREATE USER 'slave'@'%' IDENTIFIED BY 'root123456';
```
```shell
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```
### 在主机中查看主机当前状态
```shell
show master status;
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202211011444213.png)
### 另起一个终端，创建从库
```shell
mkdir -p /usr/local/mysql/mysql-slave01/{data,conf,log}
```
### 创建从库配置文件my.cnf
```shell
vim /usr/local/mysql/mysql-slave01/conf/my.cnf
```
```shell
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=2
## 指定不需要同步的数据库
binlog-ignore-db=mysql
## 开启二进制日志功能，以备slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式
binlog_format=mixed
## 二进制日志过期清理时间，默认为0，代表不自动清理
expire_logs_days=7
## 忽略主从复制中遇到的所有错误活指定的错误类型，避免slave端复制中断   
## 如：1062错误码代表主键重复；1032错误码代表主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读权限（具有super权限的用户除外）
read_only=1
```
### 启动从库
```shell
docker run -d -p 3309:3306 --privileged=true \
-v /usr/local/mysql/mysql-slave01/log:/var/log/mysql  \
-v /usr/local/mysql/mysql-slave01/data:/var/lib/mysql  \
-v /usr/local/mysql/mysql-slave01/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root123456 \
--name mysql-slave01 mysql:5.7
```
### 查看从库容器的ip并记录起来
```shell
docker inspect mysql-slave01
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202211011445629.png)
### 进入从库容器
```shell
docker exec -it mysql-slave01 /bin/bash
```
### 登录从库
```shell
mysql -uroot -proot123456 -P3309
```
### 修改一下命令对应的信息（在从库上配置主机信息）
```shell
change master to master_host='172.17.0.7',master_user='slave',master_password='root123456',master_port=3306,master_log_file='mall-mysql-bin.000003',master_log_pos=617,master_connect_retry=30;
```
### 查看从库状态
```shell
show slave status \G
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202211011453134.png)
查看Slave_IO_Running，Slave_SQL_Running两个参数，当前为no，表示暂未开始主从同步。
### 从机开启主从同步,然后再次查看从机状态发现两个参数为true，即表示配置成功
```shell
start slave;
```
```shell
show slave status \G
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202211011456238.png)