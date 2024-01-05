---
title: centos安装jdk
date: 2022-08-04 17:16:07
tags: [centos , jdk]
categories: [centos , jdk]
---
```shell
cd /usr/local
```
```shell
wget https://yoondada.oss-cn-shenzhen.aliyuncs.com/software/centos/jdk-8u381-linux-x64.tar.gz
```
```shell
tar -zxvf jdk-8u381-linux-x64.tar.gz
```
```shell
mv jdk-8u381-linux-x64 jdk
```
```shell
vim /etc/profile
```
```shell
JAVA_HOME=/usr/local/jdk
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```
```shell
:wq
```
```shell
source /etc/profile
```
```shell
java
javac
java -version
```
