---
title: CentOS7安装多版本JDK
date: 2025-01-26 09:18:55
tags:
---
# Linux 离线安装 JDK（多版本）

## 1. 准备 JDK 安装包
- 前往 [Oracle 官网](https://www.oracle.com/cn/java/technologies/downloads/#java8) 或 OpenJDK 官网，下载所需版本的 JDK（例如 JDK8、JDK11、JDK17）。
- 将下载好的压缩包上传到目标服务器。例如：
    - `jdk-8u421-linux-x64.tar.gz`
    - `jdk-11.0.26_linux-x64_bin.tar.gz`
    - `jdk-17.0.14_linux-x64_bin.tar.gz`
---

## 2. 解压 JDK
在服务器上执行以下步骤：

```bash
# 创建目录，存放 JDK
mkdir -p /usr/local/java

# 解压 JDK 到指定目录
tar -zxvf jdk-8u421-linux-x64.tar.gz -C /usr/local/java
tar -zxvf jdk-11.0.26_linux-x64_bin.tar.gz -C /usr/local/java
tar -zxvf jdk-17.0.14_linux-x64_bin.tar.gz -C /usr/local/java
```
解压完成后目录结构大致如下：

```bash
/usr/local/java/
├── jdk1.8.0_421
├── jdk-11.0.26
└── jdk-17.0.14
```
## 3. 解压 JDK
编辑配置文件：
```bash
vim /etc/profile
```
在文件末尾添加以下内容：
```bash
# 默认 JDK（例如 JDK8）
export JAVA_HOME=/usr/local/java/jdk1.8.0_421
export CLASSPATH=$JAVA_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH

# 多版本 JDK 快捷切换
alias java8='export JAVA_HOME=/usr/local/java/jdk1.8.0_421 && export CLASSPATH=.:${JAVA_HOME}/lib && export PATH=${JAVA_HOME}/bin:$PATH && java -version'
alias java11='export JAVA_HOME=/usr/local/java/jdk-11.0.26 && export CLASSPATH=.:${JAVA_HOME}/lib && export PATH=${JAVA_HOME}/bin:$PATH && java -version'
alias java17='export JAVA_HOME=/usr/local/java/jdk-17.0.14 && export CLASSPATH=.:${JAVA_HOME}/lib && export PATH=${JAVA_HOME}/bin:$PATH && java -version'
```
4. 使配置生效
```bash
   source /etc/profile
```
5. 验证安装
```bash
# 查看默认版本
java -version

# 切换到 JDK11
java11

# 切换到 JDK17
java17
```
