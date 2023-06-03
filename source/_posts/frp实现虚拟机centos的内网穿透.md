---
title: frp实现虚拟机centos的内网穿透
date: 2022-09-21 18:13:49
tags: [centos,ssh,frp]
categories: [centos,ssh,frp]
---
### 需要一台有公网ip的服务器和随便一台虚拟机centos
## 含公网ip的服务器配置如下：
### 1、下载frp
```shell
wget https://github.com/fatedier/frp/releases/download/v0.44.0/frp_0.44.0_linux_arm64.tar.gz
```
### 2、解压
```shell
tar -zxvf frp_0.44.0_linux_amd64.tar.gz
```
### 3、移动到/usr/local下吧并重命名，个人习惯
```shell
mv frp_0.44.0_linux_amd64 /usr/local/frp
```
### 4、cd到/usr/local/frp
```shell
cd /usr/local/frp/
```
### 5、删除没有的配置文件（我们这里作为服务端，把客户端相关的配置文件删除了吧）
```shell
rm -f frpc*
```
### 6、修改配置文件
```shell
vim frps.ini
```
```shell
# frps.ini
[common]
bind_port = 7000
authentication_method = token
# 认证密码，需要与客户端一致
token = 12345678
```
### 7、配置成服务的方式启动吧
```shell
sudo vi /etc/systemd/system/frps.service
```
```shell
[Unit]
Description=frps daemon
After=syslog.target  network.target
Wants=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.ini
 
[Install]
WantedBy=multi-user.target
```
### 8、一些命令
```shell
systemctl enable frps
```
##### 启动
```shell
systemctl start frps
```
##### 关闭
```shell
systemctl stop frps
```
## 客户端虚拟机配置如下：
### 1、下载frp
```shell
wget https://github.com/fatedier/frp/releases/download/v0.44.0/frp_0.44.0_linux_arm64.tar.gz
```
### 2、解压
```shell
tar -zxvf frp_0.44.0_linux_amd64.tar.gz
```
### 3、移动到/use/local下吧并重命名，个人习惯
```shell
mv frp_0.44.0_linux_amd64 /usr/local/frp
```
### 4、cd到/usr/local/frp
```shell
cd /usr/local/frp/
```
### 5、删除没有的配置文件（我们这里作为客户端，把服务端相关的配置文件删除了吧）
```shell
rm -f frps*
```
### 6、修改配置文件 frpc.ini
```shell
vim frpc.ini
```
```shell
# frpc.ini
[common]
# 对应外网服务器的ip和端口
server_addr = 43.142.62.156
server_port = 7000
authentication_method = token
# 认证密码，需要与服务端一致
token = 12345678

[ssh]
type = tcp
local_ip = 127.0.0.1
# 本机的服务端口
local_port = 22
# 让外网服务器开启的端口（需要防火墙放行）
remote_port = 12900
```
### 7、设置为启动服务
```shell
sudo vi /etc/systemd/system/frpc.service
```
```shell
[Unit]
Description=frpc daemon
After=syslog.target  network.target
Wants=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.ini
 
[Install]
WantedBy=multi-user.target
```
### 8、一些命令
```shell
systemctl enable frpc
```
##### 启动
```shell
systemctl start frpc
```
##### 关闭
```shell
systemctl stop frpc
```
