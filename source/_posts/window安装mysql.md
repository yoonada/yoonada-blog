---
title: window安装mysql
date: 2023-03-30 14:45:28
tags: [window,mysql]
categories: [window,mysql]
---
### 下载安装包
我已经将其存在oss上了

https://yoondada.oss-cn-shenzhen.aliyuncs.com/software/window/mysql-5.7.41-winx64.zip
### 移动解压重命名
E:\software\dev\mysql-5.7.41
### 新建
新建data文件夹、my.ini配置文件
### 编辑my.imi （路径是左斜杠，右斜杆会报错）
```shell
[client]
port=3306
 
[mysql]
default-character-set=utf8                
 
[mysqld]
basedir = "E:/software/dev/mysql-5.7.41"         #需要更改为自己的安装目录
datadir = "E:/software/dev/mysql-5.7.41/data"     #需要更改为自己的安装目录加上/data
port = 3306			                      
max_connections=200		                  
 
character-set-server=utf8		          
default-storage-engine=INNODB
```
### 配置环境变量-系统变量
* 新建 MYSQL_HOME，值为：E:\software\dev\mysql-5.7.41
* 系统变量中找到Path，在末尾追加：%MYSQL_HOME%\bin

### 以系统管理员打开cmd
记得是**系统管理员**
```shell
cd E:\software\dev\mysql-5.7.41\bin
```
```shell
mysqld --initialize-insecure
```
```shell
mysqld -install
```
```shell
net start mysql
```
### 设置连接密码
```shell
mysqladmin -u root password DD123456aa
```
### 搞定
```shell
mysql -u root -p
```
