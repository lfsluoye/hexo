layout: post
title: "部署django在阿里云上的服务器"
date: 2018-5-3 
categories: Python3
comments: false
tags: django
---

## centos7下部署Django（nginx+uwsgi+python3+django)
系统版本
centos7

python版本
3.6.3

django版本
1.11

uwsgi版本
2.0.15

nginx版本
1.13.10
<!-- more -->
### 进入正题，一行命令，一行注释，使用root身份登录系统执行

#### 1、安装各类基础模块
> yum gcc-c++

（为centos系统增加编译功能）

> yum install wget openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel

（安装这些模块都是为了成功编译安装python3，防止出现各种异常）
> yum install libxml*

（安装这个模块是为了让uwsig支持使用“-x"选项，能通过xml文件启动项目）
#### 2、编译安装python3
进入home路径（本人喜欢把东西都下载到这里）,执行以下命令：
> wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz

下载完成后，执行解压命令：
> tar -zxvf Python-3.6.3.tar.gz

进入解压后的Python-3.6.3文件夹，依次执行以下命令
> ./configure --prefix=/usr/local/python3

将python3安装到/usr/local/python3/路径下
> make -j2
make install -j2
ln -s /usr/local/python3/bin/python3.6 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

（以上两个ln命令是为了方便在终端中直接使用python3和pip3命令）
#### 3、给python3安装django和uwsgi以及配置启动项目的xml文件
> pip3 install django
pip3 install uwsgi

为了在终端中使用uwsgi命令，执行以下命令
> ln -s /usr/local/python3/bin/uwsgi /usr/bin/uwsgi3

将你的django项目放到你想放的路径下，例如/home/www/，假设我们的Django项目名为"myproject",里面有一个应用叫"myapp"
在你的django项目下新建 myproject.xml，内容如下：
```
<uwsgi>
    <socket>127.0.0.1:8997</socket><!-- 内部端口，自定义 -->
        <chdir>/home/www/myproject</chdir><!-- 项目路径 -->
            <module>myproject.wsgi</module>
                <processes>4</processes> <!-- 进程数 --> 
    <daemonize>uwsgi.log</daemonize><!-- 日志文件 -->
</uwsgi>
```
wq保存

#### 4、安装nginx和配置nginx.conf文件
进入home目录，执行以下命令：
> wget http://nginx.org/download/nginx-1.13.7.tar.gz

下载完成后，执行解压命令：
> tar -zxvf nginx-1.13.7.tar.gz

进入解压后的nginx-1.13.7文件夹，依次执行以下命令：
> ./configure
make
make install

nginx一般默认安装好的路径为/usr/local/nginx
在/user/local/nginx/conf/中打开nginx.conf，加入以下内容
```
server {
    listen 8996; #暴露给外部访问的端口
    server_name localhost;
        charset utf-8;
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8997; #外部访问8996就转发到内部8997
    }
    location /static/ {
        alias /home/www/myproject/myapp/static/; #项目静态路径设置
    }
}
```
（以上内容请保证在默认内容的大括号内）

wq保存后进入/usr/local/nginx/sbin/目录
执行./nginx -t命令先检查配置文件是否有错，没有错就执行以下命令：
./nginx
终端没有任何提示就证明nginx启动成功，可以通过链接查看nginx是否启动成功:
http://192.168.1.111 （请将该ip替换成你的服务器ip）
#### 5、访问项目页面
进入你的django项目路径，执行以下命令：
> uwsgi3 -x myproject.xml

以上步骤都没有出错的话，打开你的浏览器，输入以下链接，记得关闭系统防火墙或者开放8996端口
http://192.168.1.111:8996 （请将该ip替换成你的服务器ip）
网站访问成功！

