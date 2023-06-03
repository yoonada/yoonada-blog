---
title: doris-1.1.3的编译
date: 2022-10-26 17:00:21
tags: [doris]
---
### 1、创建存放doris的目录
```shell
/home/software/doris-1.1.3
```
### 2、下载doris-1.1.3源码
```shell
cd /home/software/doris-1.1.3
```
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/doris/1.1/1.1.3-rc02/apache-doris-1.1.3-src.tar.gz --no-check-certificate
```
### 3、解压
```shell
tar -zxvf apache-doris-1.1.3-src.tar.gz 
```
### 4、通过docker拉取docker-1.1.3对应的镜像版本(目前是这个)
具体查看 https://doris.apache.org/zh-CN/docs/dev/install/source-install/compilation
```shell
docker pull apache/doris:build-env-ldb-toolchain-latest
```
### 5、做两个目录映射，一个是maven的repository目录，一个是doris源码目录，避免容器挂了之后之前下载或编译的内容丢失
```shell
docker run -it -v /usr/local/maven/repository:/root/.m2 -v /home/software/doris-1.1.3/apache-doris-1.1.3-src:/root/doris-1.3.0/apache-doris-1.1.3-src apache/doris:build-env-ldb-toolchain-latest
```
### 6、查看java的版本，默认是jdk11
```shell
java -version
```
### 7、切换到jdk8吧
```shell
alternatives --set java java-1.8.0-openjdk.x86_64
```
```shell
alternatives --set javac java-1.8.0-openjdk.x86_64
```
```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0
```
```shell
java -version
```
### 8、进入到doris-1.1.3目录中
```shell
cd doris-1.3.0/apache-doris-1.1.3-src
```
### 9、开始编译 (至少需要两个小时)
```shell
sh build.sh
```