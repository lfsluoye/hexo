---
layout: post
title: "iOS--SQLite同时删除多张表写法"
date: 2016-04-25 
categories: iOS
comments: false
tags: OC
---
#### 把删除多张表的语句写在同一句sql语句里
<!-- more -->
```
 NSString *sql = [NSString stringWithFormat:
                     @"delete from LFFloors where deleteId = '%@';"
                     "delete from LFUnit where deleteId = '%@';"
                     "delete from LFLayer where deleteId = '%@';"
                     "delete from LFHouse where deleteId = '%@';",deleteId,deleteId,deleteId,deleteId];
```



