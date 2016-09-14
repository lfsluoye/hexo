---
layout: post
title: "Xcode8问题收集"
date: 2016-09-14 
categories: iOS
comments: false
tags: OC
---
### 问题一 打开相册报错
今天Xcode8 更新了,打开相册，在调用发生了崩溃现象，一开始以为是巧合，但尝试了第二次之后，似乎意识到确实出了问题，从而关注控制台，这个时候就注意到了这句Xcode给我们的忠告:
```
[access] 
This app has crashed because it attempted to access privacy-sensitive data without a usage description.  
The app's Info.plist must contain an NSPhotoLibraryUsageDescription key with a string value explaining to the user how the app uses this data.
```
<!-- more -->
不难翻译，大体意思就是这个App缺少一个获取私有(敏感)数据的权限描述，需要我们在info.plist文件中必须含有一个名字叫做NSPhotoLibraryUsageDescription的值来解释为什么应用需要使用这个数据，没错，获取相册资源的键值就是`NSPhotoLibraryUsageDescription`

感觉它”友好”的提示之后，就去plist文件中添加了下面的键值:
![在info.plist中添加key值](/assets/blogImg/添加照片key值.png)
这个时候再点击获取图片资源，就弹出了一个获取权限的问候，不会发生崩溃了

通过类似事情，说明iOS10对用户的隐私又做了进一步加强，就好像当初iOS8对定位隐私进行加强一样，作为开发者的我们貌似也是应该时刻保持这种对新知识警觉性的。
> - Privacy - Microphone Usage Description //麦克风权限
- Privacy - Contacts Usage Description   //通讯录权限
- Privacy - Camera Usage Description     //摄像头权限



