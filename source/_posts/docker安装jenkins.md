---
title: docker安装jenkins
date: 2023-11-20 16:19:45
tags: [docker, jenkins]
categories: [docker, jenkins]
---
```shell
docker run -d -u root \
    --name jenkins \
	--restart=always \
	-p 30000:8080 \
	-p 30001:50000 \
	-v jenkins-data:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:2.426.1-lts-jdk11
```

```shell
http://10.18.12.88:30000/pluginManager/advanced
```
```shell
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

```shell
sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' /var/lib/docker/volumes/jenkins-data/_data/updates/default.json
```
```shell
sed -i 's#http://www.google.com#https://www.baidu.com#g' /var/lib/docker/volumes/jenkins-data/_data/updates/default.json
```
