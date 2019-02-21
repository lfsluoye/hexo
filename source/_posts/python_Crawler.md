layout: post
title: "python-爬虫"
date: 2019-1-15 
categories: Python3
comments: false
tags: 爬虫豆瓣
---

#### 开始创建项目
```
scrapy startproject douban
```
#### 添加要爬取的网站
```
scrapy genspider douban_spider movie.douban.com
```
#### 开始爬取
```
scrapy crawl douban_spider
```
#### 数据的存储
```
scrapy crawl douban_spider -o text.json
scrapy crawl douban_spider -o text.csv
```


