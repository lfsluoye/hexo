---
layout: post
title: "iOS微信支付"
date: 2016-04-28 
categories: iOS
comments: false
tags: OC 
---

1.首先下载最新的微信支付的SDK包.[下载地址](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=11_1),拖到你的工程文件中.

简要步骤如下:
<!-- more -->
#### 1. 配置APPID

APPID要在微信开放平台申请.(让公司去注册.)

targets -> info -> URL Types 
![配置APPID](/assets/blogImg/APPID.png)
配置完是这样的
> identifier 要使用 "weixin"

![配置APPID2](/assets/blogImg/APPID2.png)
#### 2. 在appDelegate引入微信lib,和头文件. 
```
#import "WXApi.h"
#import "WXApiManager.h"
```
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {


    //向微信注册APPID
    [WXApi registerApp:@"wxb4ba3c02aa476ea1" withDescription:@"demo 2.0"];
    return YES;
}
```
#### 3. 调起微信支付
微信支付需第一次调通统一下单接口 prepayId,sign,nonceStr,timeStamp分别是微信预支付ID,签名,随机字符串,还有时间戳.
> 建议 "统一下单"由服务器端调用,然后客户端调用 支付接口.
因为 客户端生成随机字符串,还有签名,终端ip等等在客户端做并不妥当,并且一些商品的描述信息,商户的订单号还是要从服务器那边获取.

```
- (void)wxPay{

    NSString *res = [self jumpToBizPay];
    if( ![@"" isEqual:res] ){
        UIAlertView *alter = [[UIAlertView alloc] initWithTitle:@"支付失败" message:res delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];

        [alter show];
    }
}
- (NSString *)jumpToBizPay {

    //============================================================
    // V3&V4支付流程实现
    // 注意:参数配置请查看服务器端Demo
    // 更新时间：2015年11月20日
    //============================================================
//    self.wxPayURL = @"http://wxpay.weixin.qq.com/pub_v2/app/app_pay.php?plat=ios";
    //解析服务端返回json数据
    NSError *error;
    //加载一个NSURL对象
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:self.wxPayURL]];
    //将请求的url数据放到NSData对象中
    NSData *response = [NSURLConnection sendSynchronousRequest:request returningResponse:nil error:nil];
    if ( response != nil) {
        NSMutableDictionary *dict = NULL;
        //IOS5自带解析类NSJSONSerialization从response中解析出数据放到字典中
        dict = [NSJSONSerialization JSONObjectWithData:response options:NSJSONReadingMutableLeaves error:&error];
        NSLog(@"url:%@",self.wxPayURL);
        if(dict != nil){
            NSMutableString *retcode = [dict objectForKey:@"retcode"];
            if (retcode.intValue == 0){
                NSMutableString *stamp  = [dict objectForKey:@"timestamp"];

                //调起微信支付
                PayReq* req             = [[PayReq alloc] init];
                req.partnerId           = [dict objectForKey:@"partnerid"];
                req.prepayId            = [dict objectForKey:@"prepayid"];
                req.nonceStr            = [dict objectForKey:@"noncestr"];
                req.timeStamp           = stamp.intValue;
                req.package             = [dict objectForKey:@"package"];
                req.sign                = [dict objectForKey:@"sign"];
                [WXApi sendReq:req];
                //日志输出
                NSLog(@"appid=%@\npartid=%@\nprepayid=%@\nnoncestr=%@\ntimestamp=%ld\npackage=%@\nsign=%@",[dict objectForKey:@"appid"],req.partnerId,req.prepayId,req.nonceStr,(long)req.timeStamp,req.package,req.sign );
                return @"";
            }else{
                return [dict objectForKey:@"retmsg"];
            }
        }else{
            return @"服务器返回错误，未获取到json对象";
        }
    }else{
        return @"服务器返回错误";
    }
}
```
这里注意: 一大堆代码都是为了获取 到这些参数.这里的代码是微信返回的参数,实际参数我们可以从公司服务器中请求接口获取.
```
PayReq* req             = [[PayReq alloc] init];
   req.partnerId           = [dict objectForKey:@"partnerid"];
   req.prepayId            = [dict objectForKey:@"prepayid"];
   req.nonceStr            = [dict objectForKey:@"noncestr"];
   req.timeStamp           = stamp.intValue;
   req.package             = [dict objectForKey:@"package"];
   req.sign                = [dict objectForKey:@"sign"];
```

> 字段 package 暂时是固定的 @"Sign=WXPay"就行.partnerId 是微信支付分配的商户号 申请的时候就有的.固定即可

获取完成之后调用
```
[WXApi sendReq:req];
```
剩下的就只有支付结果的回调了.

请求参数列表:
![配置APPID2](/assets/blogImg/请求参数列表.png)
#### 4. 支付回调结果
a.回到 appDelegate中
遵循代理

    @interface AppDelegate () <WXApiDelegate>

b.当调用微信支付返回的时候.我们做的就是要在

    - (BOOL) application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
判断是否成功调起微信支付.
```
- (BOOL) application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{

    BOOL result = [WXApi handleOpenURL:url delegate:self]; //判断调起微信支付是否成功
    if (result) {
        return result;
    }
    return NO;
  }
```
c.实现代理方法
```
/**
 *  微信的回调. (代理方法.)
 */
-(void) onResp:(BaseResp*)resp{

    if([resp isKindOfClass:[SendMessageToWXResp class]]){
        strTitle = [NSString stringWithFormat:@"发送媒体消息结果"];
    }
    if([resp isKindOfClass:[PayResp class]]){
        //支付返回结果，实际支付结果需要去微信服务器端查询
        strTitle = [NSString stringWithFormat:@"支付结果"];

        //控制器接收通知就OK!
        switch (resp.errCode) {
            case WXSuccess:{
                strMsg = @"支付结果：成功！";
   //微信文档有提到 一定不能用客户端的返回值做标准,实际支付结果应该去服务器查询的结果为准.

                [[NSNotificationCenter defaultCenter] postNotificationName:@"WXPaySuccess" object:@"success"];
                break;
            }
            default:{
                NSLog(@"错误，retcode = %d, retstr = %@", resp.errCode,resp.errStr);
                //
                [[NSNotificationCenter defaultCenter] postNotificationName:@"WXPayFailed" object:@"fail"];
                break;
            }
        }
    }


}
```
> 支付回调结果,应该在去服务器查询.安全起见 结果一致,才显示支付成功! (有一些没说明清楚的麻烦指正.)
这里还是有一个坑的,微信会向服务器在固定的频率发送支付成功的回调,有可能 2s, 有可能3秒这样的.也就是说,支付成功之后回来的时候从服务器查询可能还残留着上一次的支付状态.(没有更新过来,需要缓几秒). 在这个频率之间去请求的话可能会出现结果不一致的情况.而且我觉得微信做的支付并不专业,用过其他几个支付平台的就知道,支付结果的回调的时候,会反回一些 有关的订单信息,或者订单状态等.以供查询,而微信只返回了一些状态码,如果还想查询其他信息话需要做另外的操作.

