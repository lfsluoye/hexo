---
layout: post
title: "iOS去除导航栏和tabbar的1px横线"
date: 2016-05-12 
categories: iOS
comments: false
tags: OC
---
#### 1.在自己定义的导航栏中或者设计稿中经常需要去除导航栏的1px横线，主要是颜色太不协调了
![去除前导航栏](/assets/blogImg/去除前导航栏.png)
要去除这1px的横线，首先应该知道它是什么，在Xcode的界面调试中可以看到，它其实是UIImageView来的
<!-- more -->
![去除前导航栏](/assets/blogImg/找到横线是什么了.png)
其实这是navigationBar的shadowImage，所以只要设置它为空即可，但是设置它为空之前应该先设置它的背景也为空，全部代码如下：
```
[self.navigationController.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];
[self.navigationController.navigationBar setShadowImage:[UIImage new]];
```
![去除前导航栏](/assets/blogImg/完成之后的效果了.png)
既然导航栏的那一横线能去除，那tabbar那一横线也是能去除的了（其实也是shadowImage来的）···
#### 方法一：
自定义UITabBarController
#### 方法二：
```
[self.tabBarController.tabBar setBackgroundImage:[UIImage new]];
[self.tabBarController.tabBar setShadowImage:[UIImage new]];
```
反之，如果我们想自定义那一横线的颜色也是可以的，只要设置它的shadowImage即可。


