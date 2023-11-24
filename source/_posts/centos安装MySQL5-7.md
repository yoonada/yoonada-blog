---
title: centos安装MySQL5.7
date: 2023-11-24 08:52:58
tags: [centos, MySQL]
categories: [centos, MySQL]
---
```shell
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```
```shell
yum -y install mysql57-community-release-el7-10.noarch.rpm
```
```shell
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```
```shell
yum -y install mysql-community-server
```
```shell
systemctl start mysqld
```
```shell
systemctl status mysqld
```
```shell
grep "password" /var/log/mysqld.log
```
```shell
mysql -uroot -p
```
```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'DD@123456!';
```
```shell
grant all privileges on *.* to 'root'@'%' identified by 'DD@123456!' with grant option;
```
```shell
flush privileges;
```
