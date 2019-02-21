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
3. 创建数据表和迁移生成静态文件
python3 manage.py makemigrations --empty 你的应用名 (失败时)
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py collectstatic
4. 创建后台管理系统
python3 manage.py createsuperuser

5. 阿里云远程连接密码:627443
<!-- more -->
#### 常用命令
>- python3 manage.py runserver
>- cd /usr/local/nginx/conf/
>- cd /usr/local/nginx/sbin/
>- ./nginx -t
>- ./nginx
>- cd /home/www/codetimeproject/
>- uwsgi3 -x codetimeproject.xml
>- uwsgi3 --reload codetimeproject.xml
>- python3 manage.py collectstatic 
>- mysql -uroot -p
>- uwsgi3 --ini uwsgi_codetime.ini&/usr/local/nginx/sbin/nginx
#####启动MySQL服务
>- systemctl start mysqld
#####查看MySQL服务
>- systemctl status mysqld.service
#### 查看所有端口
>- netstat -ntlp



