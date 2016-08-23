---
layout: post
title: "iOS二维码生成(带logo)"
date: 2015-1-30 
comments: false
tags: ios 
---
制作一个中心带有logo的二维码
<!-- more -->
```OC
/
//  ViewController.m
//  内置图片二维码
//
//  Created by hhq on 16/7/15.
//  Copyright © 2016年 com.baiduniang. All rights reserved.
//

#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong) UIImageView * imageView;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    //二维码生成 实质:  把字符串转变为 图片
    // 需要 coreImage框架, 已经包含在了 UIKit框架里面

    [self logoQrCode];
}
//MARK: 二维码中间内置图片,可以是公司logo
-(void)logoQrCode{

    //
    NSArray *filters = [CIFilter filterNamesInCategory:kCICategoryBuiltIn];
    NSLog(@"%@",filters);

    //二维码过滤器
    CIFilter *qrImageFilter = [CIFilter filterWithName:@"CIQRCodeGenerator"];

    //设置过滤器默认属性 (老油条)
    [qrImageFilter setDefaults];

    //将字符串转换成 NSdata (虽然二维码本质上是 字符串,但是这里需要转换,不转换就崩溃)
    NSData *qrImageData = [@"宁宁,我爱你" dataUsingEncoding:NSUTF8StringEncoding];

    //我们可以打印,看过滤器的 输入属性.这样我们才知道给谁赋值
    NSLog(@"%@",qrImageFilter.inputKeys);
    /*
     inputMessage,        //二维码输入信息
     inputCorrectionLevel //二维码错误的等级,就是容错率
     */


    //设置过滤器的 输入值  ,KVC赋值
    [qrImageFilter setValue:qrImageData forKey:@"inputMessage"];

    //取出图片
    CIImage *qrImage = [qrImageFilter outputImage];

    //但是图片 发现有的小 (27,27),我们需要放大..我们进去CIImage 内部看属性
    qrImage = [qrImage imageByApplyingTransform:CGAffineTransformMakeScale(20, 20)];

    //转成 UI的 类型
    UIImage *qrUIImage = [UIImage imageWithCIImage:qrImage];


    //----------------给 二维码 中间增加一个 自定义图片----------------
    //开启绘图,获取图形上下文  (上下文的大小,就是二维码的大小)
    UIGraphicsBeginImageContext(qrUIImage.size);

    //把二维码图片画上去. (这里是以,图形上下文,左上角为 (0,0)点)
    [qrUIImage drawInRect:CGRectMake(0, 0, qrUIImage.size.width, qrUIImage.size.height)];


    //再把小图片画上去
    UIImage *sImage = [UIImage imageNamed:@"Snip20160715_4"];

    //
    //    UIImageView *sImageView = [[UIImageView alloc]initWithFrame:CGRectMake(50, 50, 60, 60)];
    //    [self.view addSubview:sImageView];
    //    sImageView.image = sImage;
    //    // 类似于clip，使用masksToBounds阴影效果无效
    //    sImageView.layer.masksToBounds = YES;
    //    //图层的圆角半径
    //    sImageView.layer.cornerRadius = 50;



    CGFloat sImageW = 100;
    CGFloat sImageH= sImageW;
    CGFloat sImageX = (qrUIImage.size.width - sImageW) * 0.5;
    CGFloat sImgaeY = (qrUIImage.size.height - sImageH) * 0.5;

    [sImage drawInRect:CGRectMake(sImageX, sImgaeY, sImageW, sImageH)];

    //获取当前画得的这张图片
    UIImage *finalyImage = UIGraphicsGetImageFromCurrentImageContext();

    //关闭图形上下文
    UIGraphicsEndImageContext();



    //设置图片
    self.imageView.image = finalyImage;
}

-(UIImageView *)imageView{
    if(_imageView == nil){

        _imageView = [[UIImageView alloc]initWithFrame:CGRectMake(0, 50, 400, 400)];
        [self.view addSubview:_imageView];
    }
    return _imageView;
}
@end
```
![带logo二维码](/assets/blogImg/QrCode.png) 



