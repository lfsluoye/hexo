---
layout: post
title: "iOS不常用但是实用"
date: 2016-08-25 
categories: iOS
comments: false
tags: OC
---
### 随时更新
#### 1. 获取控件在屏幕上的位置
<!-- more -->
```
@interface LFAddressView()

@property (nonatomic, assign) CGRect rectAddress;

@end
//.m
/**
     *  获取控件在屏幕上的位置
     */
    UIWindow * window=[[[UIApplication sharedApplication] delegate] window];
    self.rectAddress = [btn convertRect: btn.bounds toView:window];
```



