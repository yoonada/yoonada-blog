---
title: Django写CRUD
date: 2023-08-30 14:26:44
tags: [Python,Django]
categories: [Python,Django]
---

# 基于Django写CRUD接口
## 安装Django相关环境

* 安装Django

```shell
pip install django
```

* 创建Django项目

    * 使用命令行创建（已加入环境变量）

      ```shell
      django-admin startproject django_crud_restful
      ```

      如果忘记配置环境变量，请配置上，教程如下：

      1、先找一下Python的安装路径

      ```shell
      pip -V
      ```

      ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251659545.png)

      2、在系统变量的Path最后，追加Python的Scripts路径

      ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251701823.png)

      ```shell
      C:\Users\YoonaDa\AppData\Local\Programs\Python\Python311\Scripts\
      ```

    * 使用PyCharm创建

      ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251707613.png)

* 安装 Django REST framework

```shell
pip install djangorestframework
```

## 新增/修改配置

### 新建应用

* 新建应用

  ```shell
  cd C:\PycharmProjects\django_crud_restful
  ```

  ```shell
  python manage.py startapp api
  ```

  ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251718648.png)

* 修改settings.py（INSTALLED_APPS中加入刚刚新建的应用的主配置类的路径）

  ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251743958.png)

### 连接MySQL

* 首先，自己的MySQL（**建议8.x版本**）创建一下数据库，比如：django_crud_restful

* 创建一张测试表吧

  ```sql
  CREATE TABLE `sys_user` (
    `id` bigint NOT NULL AUTO_INCREMENT COMMENT '自增id',
    `username` varchar(60) COLLATE utf8mb4_general_ci NOT NULL COMMENT '用户名',
    `password` varchar(60) COLLATE utf8mb4_general_ci NOT NULL COMMENT '密码',
    `phone` varchar(20) COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '手机号码',
    `age` int DEFAULT NULL COMMENT '年龄',
    `sex` int DEFAULT '0' COMMENT '性别：0、未知；1、男；2、女；默认为：0',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    `update_time` datetime DEFAULT NULL COMMENT '更新时间',
    `is_delete` int DEFAULT '0' COMMENT '是否删除：0、未删除；1、删除；默认为：0',
    PRIMARY KEY (`id`),
    UNIQUE KEY `unique_username_index` (`username`) USING BTREE
  ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='系统用户表';
  ```

* 命令行安装pymysql
  ```shell
  pip install pymysql
  ```

* 修改`settings.py`

    * 默认是sqlite3，注释掉

      ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251733334.png)

    * 修改为MySQL

      ```shell
      DATABASES = {
          "default": {
              "ENGINE": "django.db.backends.mysql",
              "NAME": "django_crud_restful",
              "USER": "root",
              "PASSWORD": "DD123456aa",
              "HOST": "43.142.62.156",
              "PORT": "3336",
          }
      }
      ```

* 修改项目文件夹下的`__init__.py`文件

  ![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251739099.png)

  ``` python
  import pymysql
  
  pymysql.install_as_MySQLdb()
  ```

* 执行迁移命令

  ```python
  python manage.py makemigrations
  ```

  ```python
  python manage.py migrate
  ```

## 编写接口

### 逆向工程生成实体

* 执行生成命令，生成到db.py中

```shell
python manage.py inspectdb > db.py
```

* 拷贝我们的测试表到model.py中

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251759381.png)

### 安装 Django REST framework

* 执行如下命令：

```python
pip install djangorestframework
```

* 在项目的 `settings.py` 文件中，将 `'rest_framework'` 添加到 `INSTALLED_APPS` 列表

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251805607.png)

* 新建`serializers.py`

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251808356.png)

```python
from rest_framework import serializers
from .models import SysUser

class SysUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = SysUser
        fields = '__all__'
```

* 修改`views.py`

```python
from django.shortcuts import render

# Create your views here.
from rest_framework import viewsets
from .models import SysUser
from .serializers import SysUserSerializer

class SysUserViewSet(viewsets.ModelViewSet):
    queryset = SysUser.objects.all()
    serializer_class = SysUserSerializer
```

* 修改`url.py`

```python
from django.urls import path, include

from rest_framework.routers import DefaultRouter
from api.views import SysUserViewSet

router = DefaultRouter()
router.register(r'users', SysUserViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251812430.png)

### 启动服务

http://127.0.0.1:8000/api/users/

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/202308251815618.png)