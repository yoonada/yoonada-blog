---
title: docker安装xxl-job
date: 2024-01-15 16:27:25
tags: [docker , xxl-job]
categories: [docker , xxl-job]
---
#### 建库sql
```shell
https://github.com/xuxueli/xxl-job/blob/2.4.0/doc/db/tables_xxl_job.sql
```
#### 安装
```shell
docker pull xuxueli/xxl-job-admin:2.4.0
```
```shell
mkdir -p /logs/docker
```
```shell
docker run -e PARAMS="--spring.datasource.username=root --spring.datasource.password=xxx --spring.datasource.url=jdbc:mysql://xxx:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" -p 35000:8080 -v /logs/docker:/data/applogs --name xxl-job-admin -d xuxueli/xxl-job-admin:2.4.0
```
```shell
http://xxx:35000/xxl-job-admin/
```
