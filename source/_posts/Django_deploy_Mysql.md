layout: post
title: "centos7上安装MySQL数据库"
date: 2018-5-5 
categories: Python3
comments: false
tags: django
---

###  在 CentOS7 上安装 MySQL5.7

1. 通过 SecureCRT 连接到阿里云 CentOS7 服务器；
2. 进入到目录 /usr/local/ 中：
cd /usr/local/
3. 创建目录 /usr/local/tools，如果有则忽略： 
mkdir -p tools
4. 创建 /usr/local/mysql 目录，如果已存在则忽略：
mkdir -p mysql
5. 进入到目录 /usr/local/tools 中：
cd tools/
6. 查看系统中是否已安装 MySQL 服务：
rpm -qa | grep mysql
或
yum list installed | grep mysql
<!-- more -->
7. 如果已安装则删除 MySQL 及其依赖的包：
yum -y remove mysql-libs.x86_64
8. 下载 mysql57-community-release-el7-8.noarch.rpm 的 YUM 源：
wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
9. 安装 mysql57-community-release-el7-8.noarch.rpm：
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
安装完后，得到如下两个包：
mysql-community.repo
mysql-community-source.repo
10. 安装 MySQL：
yum install mysql-server
遇到疑问的地方，一路 Y 下去即可；
安装完毕后，在  /var/log/mysqld.log 文件中会自动生成一个随机的密码，我们需要先取得这个随机密码，以用于登录 MySQL 服务端：
grep "password" /var/log/mysqld.log
打印如下内容：
A temporary password is generated for root@localhost: hilX0U!9i3_6
我们复制 root@localhost: 后面的随机字符串，这个字符串就是 MySQL 在安装完成后为我们随机生成的密码；
11. 登录到 MySQL 服务端并更新用户 root 的密码：
mysql -u root -philX0U!9i3_6
打印出 MySQL 的版本即表明已登录；
设置用户 root 可以在任意 IP 下被访问：
grant all privileges on *.* to root@"%" identified by "新密码";
设置用户 root 可以在本地被访问：
grant all privileges on *.* to root@"localhost" identified by "新密码";
刷新权限使之生效：
flush privileges;
更新 MySQL 的用户 root的密码：
set password = password('新密码'); 
注意：由于 MySQL5.7 采用了密码强度验证插件 validate_password，故此我们需要设置一个有一定强度的密码；
输入 exit 后用新密码再次登录看看吧！
12. 查看 MySQL 当前都内置了哪些数据库：
mysql> show databases;
我们发现其内置了如下一些数据库：
information_schema
mysql              
performance_schema
sys 
13. 启动 MySQL 服务：
service mysqld start
14. 关闭 MySQL 服务：
service mysqld stop
15. 重启 MySQL 服务：
service mysqld restart
16. 查看 MySQL 的状态：
service mysqld status
17. 查看 MySQL 的字符集：
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client    | utf8                      |
| character_set_connection | utf8                      |
| character_set_database  | latin1                    |
| character_set_filesystem | binary                    |
| character_set_results    | utf8                      |
| character_set_server    | latin1                    |
| character_set_system    | utf8                      |
| character_sets_dir      | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
查看指定的数据库中指定数据表的字符集，如查看 mysql 数据库中 servers 表的字符集：
  show table status from mysql like '%servers%';
查看指定数据库中指定表的全部列的字符集，如查看 mysql 数据库中 servers 表的全部的列的字符集：
show full columns from servers;
18. 设置 MySQL 的字符集为 UTF-8：
打开 /etc 目录下的 my.cnf 文件（此文件是 MySQL 的主配置文件）：
/etc/my.cnf
在 [mysqld] 前添加如下代码：
[client]
default-character-set=utf8
在 [mysqld] 后添加如下代码：
character_set_server=utf8
再次查看字符集：
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client    | utf8                      |
| character_set_connection | utf8                      |
| character_set_database  | utf8                      |
| character_set_filesystem | binary                    |
| character_set_results    | utf8                      |
| character_set_server    | utf8                      |
| character_set_system    | utf8                      |
| character_sets_dir      | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
19. /var/lib/mysql 是存放数据库文件的目录
20. /var/log 目录下的 mysqld.log 文件记录 MySQL 的日志；
21. MySQL 采用的 TCP/IP 协议传输数据，默认端口号为 3306，我们可以通过如下命令查看：
netstat -anp
22. 忘记密码时，可用如下方法重置：
```
# service mysqld stop
# mysqld_safe --user=root --skip-grant-tables --skip-networking &
# mysql -u root 
mysql> use mysql;
mysql> update user set password=password("new_password") where user="root"; 
mysql> flush privileges;
```

