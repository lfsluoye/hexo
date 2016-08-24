---
layout: post
title: "iOS开发--搞定烦躁的验证码"
date: 2016-7-11 
categories: iOS
comments: false
tags: OC 
---

我们开发的app里面几乎都有登陆注册系统，这时候就少不了验证码的发挥了。每次都得写这些重复的代码，没有营养又不得不写。今天下班时间，将验证码这一功能封装一下。
准备：Mob -- SDK
我们公司采用的短信验证系统时候Mob的SDK。
所以下面代码出现的网络请求都是Mob SDK 中的方法。
首先看下Mob 给的代码示例：
<!-- more -->
```
/**
 *  @from                    v1.1.1
 *  @brief                   获取验证码(Get verification code)
 *
 *  @param method            获取验证码的方法(The method of getting verificationCode)
 *  @param phoneNumber       电话号码(The phone number)
 *  @param zone              区域号，不要加"+"号(Area code)
 *  @param customIdentifier  自定义短信模板标识 该标识需从官网http://www.mob.com上申请，审核通过后获得。(Custom model of SMS.  The identifier can get it  from http://www.mob.com  when the application had approved)
 *  @param result            请求结果回调(Results of the request)
 */

[SMSSDK getVerificationCodeByMethod:SMSGetCodeMethodSMS phoneNumber:@"159****1689"
                                                               zone:@"86"
                                                   customIdentifier:nil
                                                             result:^(NSError *error){
       if (!error) {
            NSLog(@"获取验证码成功");
        } else {
            NSLog(@"错误信息：%@",error);
        }];




[SMSSDK commitVerificationCode:self.verifyCodeField.text phoneNumber:_phone zone:_areaCode result:^(NSError *error) {

            if (!error) {
                NSLog(@"验证成功");
            }
            else
            {
                NSLog(@"错误信息:%@",error);
            }
        }];
```
好了，准备活动做完了，我们来理一理这个验证码怎么做。

准备的类：UIButton NSTimer
其实实现这个功能有很多方法，思路也是很清楚
在用户 按下 获取验证码的时候，令button 的title 倒计时改变（NSTimer）并令button userInteractionEnabled = NO 当计时结束的时候 释放计时器，并且button的title改成原本的样子和userInteractionEnabled = YES
实现起来很简单。
不过我们怎么封装这个验证码的方法呢。
1.自定义button类
2.使用button的Category 来实现封装这个功能。（楼主 采用这种方法）
虽然在button 的 结构里搞了一些网络请求，有点不合理。但整体上，实际效果是不错的。

我们来看下具体的操作
h文件

```
#import <UIKit/UIKit.h>

typedef void(^handelBlock)(NSError *error);
@interface UIButton (VerificationCode)

//获取验证码
- (void)getCode:(NSString *)phone
          block:(handelBlock)block;
//验证验证码
- (void)VerificationCode:(NSString *)phone
                    Code:(NSString *)code
                   block:(handelBlock)block;
@end
```
m文件
```
import "UIButton+VerificationCode.h"
#import <SMS_SDK/SMSSDK.h>
#import <objc/runtime.h>

const NSString *beginTimeKey;
const NSString *timerKey;
@implementation UIButton (VerificationCode)

#pragma mark --  public
//获取验证码
- (void)getCode:(NSString *)phone
          block:(handelBlock)block {
    self.userInteractionEnabled = NO;
    [SMSSDK getVerificationCodeByMethod:SMSGetCodeMethodSMS phoneNumber:phone zone:@"86" customIdentifier:nil result:^(NSError *error) {
        block(error);
    }];
    [self configureTimer];

}
//验证验证码
- (void)VerificationCode:(NSString *)phone
                    Code:(NSString *)code
                   block:(handelBlock)block {
    [SMSSDK commitVerificationCode:code  phoneNumber:phone zone:@"86" result:^(NSError *error) {
        block(error);
    }];

}

#pragma mark 计时器
- (void)configureTimer {
    NSInteger beginTime = 60;

    NSTimer *timer =[NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timeClock) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

    [self setTimer:timer];
    [self setBeginTime:beginTime];

}
- (void)timeClock {
    [self setBeginTime:([self beginTime]-1)];
    NSInteger beginTime = [self beginTime];
    NSTimer *timer = [self timer];
    if (beginTime == 0) {
        [timer invalidate];
        self.userInteractionEnabled = YES;
        [self setTitle:@"获取验证码" forState:UIControlStateNormal];

    }else {
        self.userInteractionEnabled = NO;
        [self setTitle:[NSString stringWithFormat:@"%lds",(long)beginTime] forState:UIControlStateNormal];
    }


}
#pragma mark -- set get
- (NSInteger)beginTime {
    return [objc_getAssociatedObject(self, &beginTimeKey) integerValue];
}
- (void)setBeginTime:(NSInteger)beginTime {
    objc_setAssociatedObject(self, &beginTimeKey, @(beginTime), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (NSTimer *)timer {
    return objc_getAssociatedObject(self, &timerKey);
}
- (void)setTimer:(NSTimer *)timer {
     objc_setAssociatedObject(self, &timerKey, timer, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
```
这边验证验证码的方法放在button的类目里面，有点不是很合理。但如果分散了其他代码里面，又很恶心。多以索性就一起放在这个类目里面。用起来挺爽。
        ![验证码](/assets/blogImg/验证码.gif)


