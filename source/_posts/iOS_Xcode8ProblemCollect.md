---
layout: post
title: "Xcode8问题收集"
date: 2016-09-14 
categories: iOS
comments: false
tags: Xcode8
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
>- NSBluetoothPeripheralUsageDescription //访问蓝牙
- NSCalendarsUsageDescription //访问日历
- NSCameraUsageDescription //相机
- NSContactsUsageDescription //通讯录
- NSHealthShareUsageDescription // 访问健康分享
- NSHealthUpdateUsageDescription // 访问健康更新
- NSHomeKitUsageDescription //HomeKit
- NSLocationAlwaysUsageDescription // 始终访问位置
- NSLocationWhenInUseUsageDescription //在使用期间访问位置
- NSMicrophoneUsageDescription // 麦克风
- NSMotionUsageDescription // 访问运动与健身
- NSPhotoLibraryUsageDescription // 相册
- NSRemindersUsageDescription // 访问提醒事项
- NSSiriUsageDescription // Siri
- NSSpeechRecognitionUsageDescription //语音识别
- NSVideoSubscriberAccountUsageDescription // 视频这块的认证
- NSVoIPUsageDescription // VoIP通话

### 问题二 快捷键不灵
像我最常用的 注释 "Command" + "/" 居然不管用啦
**启动终端输入下面这句话，然后重启电脑就好啦**
> ~ sudo /usr/libexec/xpccachectl

### 问题三 模拟器的选项不见了
很奇怪的感觉，公司的电脑更新没有出现这个问题，回来后个人电脑出现啦。
一个最直接的方法，手动添加。直接到 Windows --> Devices, 看到左下角添加模拟器。

### 问题四 打印时出现一大堆信息
一堆很奇怪的信息，暂时也不知什么情况。
![关闭无用信息](/assets/blogImg/关闭无用信息.png)
在 Edit Scheme 中 ，如上图设置 `OS_ACTIVITY_MODE : disable`, 然后就 OK 啦。

### 问题五 隐藏状态栏的功能坏掉了
升级到 iOS 10.0后，在查看全屏图片的时候，需要在 Present 之前给要 present 的 view controller 设置 modalPresentationCapturesStatusBarAppearance = true。然后就好啦
```
TestViewController *testVC = [[TestViewController alloc] init];
testVC.modalPresentationCapturesStatusBarAppearance = true;
[self presentViewController:testVC animated:YES completion:nil];
```

### 问题六 解决xcode8中的xib冲突
最近xCode8 和 iOS10 相继出来，团队中有人使用xcode8 来开发工程；
他们提交代码后，我们发现使用xcode8 修改的xib文件，在xcode7，7.3.1上无法运行:

解决方法如下：
1. 右键点击该xib文件，使用source code查看；
2. 在source code 中去掉下面这行代码：
`<capability name="documents saved in the Xcode 8 format" minToolsVersion="8.0"/>`
3. clean 一下，即可成功运行。

