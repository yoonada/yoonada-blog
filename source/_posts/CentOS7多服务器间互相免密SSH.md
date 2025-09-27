---
title: CentOS7多服务器间互相免密SSH
date: 2025-08-08 08:08:08
tags: [CentOS7,SSH]
categories: [CentOS7,SSH]
---
# CentOS7多服务器间互相免密SSH

## 场景说明
三台服务器之间需要互相免密登录。服务器信息如下：

- **10** → `10.18.12.10`
- **11** → `10.18.12.11`
- **12** → `10.18.12.12`

---

## 一、生成 SSH 密钥对
在三台服务器上分别执行以下命令：

```bash
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
```
## 二、分发公钥到目标服务器
1. 在 10 服务器上执行
```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.11
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.12
```
2. 在 11 服务器上执行
```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.10
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.12
```
3. 在 12 服务器上执行
```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.10
   ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.18.12.11
```
这样，三台服务器就完成了互相免密登录的配置。
## 三、检查 SSH 目录和文件权限
在三台服务器上分别执行以下命令，确保权限正确：
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```
## 四、验证免密登录

以 10 服务器 为例，测试免密登录到其他服务器：

### 登录到 11
```bash
ssh root@10.18.12.11
```
### 登录到 12
```bash
ssh root@10.18.12.12
```
如果能直接进入对方服务器，而无需输入密码，即表示配置成功。