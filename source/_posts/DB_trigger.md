layout: post
title: "触发器"
date: 2018-5-8 
categories: Python3
comments: false
tags: 数据库
---
#### 创建触发器语法
```
DELIMITER // #改变结束符号
create trigger triggerName
after/before insert/update/delete on 表名
for each row #这句话是固定的
BEGIN
sql语句
END
DELIMITER ;
```
<!-- more -->
#### 删除触发器的语法
```
drop trigger 触发器名
```
#### 查看触发器
```
show triggers
```

#### 如何在触发器引用行的值
对于insert而言，新增的行用new来表示
行中的每一列的值，用new.列名来表示

对于delete而言，新增的行用old来表示
行中的每一列的值，用old.列名来表示

#### 例子
```
DELIMITER //
create trigger tr_order_number
before insert on codetime_product
for each row 
BEGIN
declare n int;
select IFNULL(max(right(text3,4)),0) into n from codetime_product where mid(text3,1,8)=DATE_FORMAT(CURDATE(),'%Y%m%d');
set NEW.text3=concat(DATE_FORMAT(CURDATE(),'%Y%m%d'),right(10001+n,4));
END;
//
DELIMITER ;
```



