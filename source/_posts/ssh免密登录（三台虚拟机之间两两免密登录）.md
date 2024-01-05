---
title: ssh免密登录（三台虚拟机之间两两免密登录）
date: 2022-09-20 14:15:40
tags: [CentOS7,ssh]
categories: [CentOS7,ssh]
---
# 现有三台CentOS7的虚拟机，要求两两之间实现免密登录
192.168.118.128
192.168.118.129
192.168.118.130
### 1、三台虚拟机都执行以下命令：
```shell
ssh-keygen -t rsa
```
### 2、三台虚拟机都进入/.ssh/目录（该目录存放秘钥对）
```shell
cd ~/.ssh/
```
### 3、三台虚拟机都执行以下命令：
```shell
touch authorized_keys
```
```shell
chmod 600 authorized_keys
```
```shell
cat id_rsa.pub >> authorized_keys
```
#### 此时，三个虚拟机的当前目录都为.ssh下
### 4、把129的公钥追加到128的authorized_keys
```shell
scp id_rsa.pub 192.168.118.128:/home/
```
#### 点击切换到128的连接，在128上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 把128的公钥追加到129的authorized_keys
```shell
scp id_rsa.pub 192.168.118.129:/home/
```
#### 点击切换到129的连接，在129上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 此时，128、129两两之间可实现ssh密码登录，接下来128与130之间以及129与130之间类似
### 5、把130的公钥追加到128的authorized_keys  (130上)
```shell
scp id_rsa.pub 192.168.118.128:/home/
```
#### 点击切换到128的连接，在128上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 把128的公钥追加到130的authorized_keys 
```shell
scp id_rsa.pub 192.168.118.130:/home/
```
#### 点击切换到130的连接，在130上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 此时实现了128与130之间的免密

### 6、把130的公钥追加到129的authorized_keys (130上)
```shell
scp id_rsa.pub 192.168.118.129:/home/
```
#### 点击切换到129的连接，在129上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 把129的公钥追加到130的authorized_keys
```shell
scp id_rsa.pub 192.168.118.130:/home/
```
#### 点击切换到130的连接，在130上执行如下命令
```shell
cat ../../home/id_rsa.pub >> ./authorized_keys
```
#### 此时实现了129与130之间的免密
### 6、两两之间校验
```shell
ssh root@192.168.118.128
```
```shell
ssh root@192.168.118.129
```
```shell
ssh root@192.168.118.130
```
