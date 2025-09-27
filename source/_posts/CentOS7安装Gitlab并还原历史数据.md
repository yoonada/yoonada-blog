---
title: CentOS7安装Gitlab并还原历史数据
date: 2025-05-20 09:34:50
tags: [CentOS7,Gitlab]
categories: [CentOS7,Gitlab]
---
### 1. **安装 GitLab**

#### 1.1 更新并安装依赖包

在新服务器上执行以下命令，更新系统并安装必要的依赖包：

```shell
sudo yum update -y
sudo yum install -y curl policycoreutils perl
```

#### 1.2 安装 OpenSSH 服务

安装并启动 SSH 服务，以便远程管理：

```shell
sudo yum install -y openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
```

#### 1.3 安装 Postfix（可选）

安装并启用 Postfix 邮件服务（如果 GitLab 需要发送邮件）：

```shell
sudo yum install -y postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

#### 1.4 添加 GitLab 仓库并安装 GitLab

从 GitLab 官方仓库下载并安装 GitLab CE（社区版）：

```shell
wget https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-16.5.1-ce.0.el7.x86_64.rpm
sudo yum install gitlab-ce-16.5.1-ce.0.el7.x86_64.rpm
```

#### 1.5 配置 GitLab

编辑配置文件：

```shell
sudo vi /etc/gitlab/gitlab.rb
```

找到：

```shell
external_url 'http://gitlab.example.com'
```

修改为你的 **IP+端口**，例如：

```shell
external_url 'http://xxx.xxx.xxx.xxx:xxxx'
```

保存后执行：

```shell
sudo gitlab-ctl reconfigure
```

#### 1.6 开放防火墙端口（如果有防火墙）

如果需要，开放 GitLab 服务的端口（例如 2701）：

```shell
sudo firewall-cmd --permanent --add-port=2701/tcp
sudo firewall-cmd --reload
```

###  2. 恢复关键配置文件

**必须从旧服务器拷贝过来的两个文件：**

- `/etc/gitlab/gitlab.rb`
- `/etc/gitlab/gitlab-secrets.json`

假设它们在新服务器 `/usr/local/gitlab/old/` 目录：

```shell
# 替换文件
sudo cp /usr/local/gitlab/old/gitlab.rb /etc/gitlab/gitlab.rb
sudo cp /usr/local/gitlab/old/gitlab-secrets.json /etc/gitlab/gitlab-secrets.json

# 设置权限
sudo chown root:root /etc/gitlab/gitlab.rb
sudo chown root:root /etc/gitlab/gitlab-secrets.json
sudo chmod 600 /etc/gitlab/gitlab.rb
sudo chmod 600 /etc/gitlab/gitlab-secrets.json
```

### 3. 重新配置 GitLab（应用配置文件）

```shell
sudo gitlab-ctl reconfigure
```

### 4. **启动 PostgreSQL（如果未自动启动）**

确认数据库服务是否启动：

```shell
sudo gitlab-ctl status
```

如果 PostgreSQL 没启动，可以执行：

```shell
sudo gitlab-ctl start postgresql
```

------

### 5. 准备备份文件

将旧服务器的备份文件复制到新服务器：

默认存放目录是：

```shell
/var/opt/gitlab/backups/
```

例如：

```shell
1754527017_2025_08_07_16.5.1_gitlab_backup.tar
```

确认文件已存在：

```shell
ls -lh /var/opt/gitlab/backups/
```

------

### 6. 执行还原

1. **停止相关服务（避免数据写入冲突）**

```shell
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
```

1. **执行还原命令**

```shell
sudo gitlab-rake gitlab:backup:restore BACKUP=1754527017_2025_08_07_16.5.1
```

> 注意：只写数字时间戳部分即可（不要带 `.tar` 后缀）。

1. **还原完成后重启服务**

```shell
sudo gitlab-ctl start
```

------

### 7. 验证 GitLab

1. 打开浏览器访问：

```shell
http://xxx.xxx.xxx.xxx:2701
```

1. 登录并确认项目、用户、代码仓库等是否恢复正常。

------

### 8. 额外提示

- 如果 GitLab 页面打不开，可以看日志：

```shell
  sudo gitlab-ctl tail
  ```

- 如果数据库连接错误，先确认 PostgreSQL 服务启动。

- 如果上传文件、头像、CI 缓存等缺失，需要确认 **uploads、artifacts、packages** 等目录是否随备份恢复。
