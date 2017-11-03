layout: post
title: "python-django"
date: 2016-12-23 
categories: Python3
comments: false
tags: django
---
### django 入门和实践
1. 创建项目
django-admin startproject 项目名
2. 创建应用
python3 manage.py startapp 应用名
添加应用名到settings.py中的INSTALLED_APPS里
3. 创建数据表和迁移
python3 manage.py makemigrations
python3 manage.py migrate
4 创建后台管理系统
python3 manage.py createsuperuser

#### 常用命令
>- python3 manage.py runserver



