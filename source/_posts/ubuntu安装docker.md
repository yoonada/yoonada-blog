---
title: ubuntu安装docker
date: 2023-12-26 14:36:00
tags: [ubuntu,docker]
categories: [ubuntu,docker]
---
```shell
sudo apt-get remove docker \
               docker-engine \
               docker.io
```

```shell
sudo apt-get update
```
```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
```shell
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```shell
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```shell
sudo apt-get update
```
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
```shell
sudo systemctl enable docker
```
```shell
sudo systemctl start docker
```
```shell
sudo groupadd docker
```
```shell
sudo usermod -aG docker $USER
```
退出当前终端并重新登录
```shell
sudo mkdir -p /etc/docker
```
```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9w9zqgnf.mirror.aliyuncs.com"]
}
EOF
```
```shell
sudo systemctl daemon-reload
```
```shell
sudo systemctl restart docker
```
