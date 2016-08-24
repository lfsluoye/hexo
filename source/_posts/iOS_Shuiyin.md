---
layout: post
title: "iOS实现刮刮乐功能与图片添加水印功能"
date: 2016-07-05 
categories: iOS
comments: false
tags: OC 
---
代码简单直接上代码
**1.创建子视图**
```
- (void)setupSubviews {
    UILabel *label = [[UILabel alloc]initWithFrame:CGRectMake(20, 100, kScreenWidth-40, kScreenWidth-40)];
    label.text = @"斯塔克家族 - 凛冬将至";
    label.numberOfLines = 0;
    label.backgroundColor = [UIColor grayColor];
    label.font = [UIFont systemFontOfSize:30];
    label.textAlignment = NSTextAlignmentCenter;
    [self.view addSubview:label];
    self.imageView = [[UIImageView alloc]initWithFrame:CGRectMake(20, 100, kScreenWidth-40, kScreenWidth-40)];
    self.imageView.image = [UIImage imageNamed:@"001"];
    [self.view addSubview:self.imageView ];
    //图片添加文字水印
    self.imageView.image = [self image:_imageView.image addText:@"哪个家族的家徽?" msakRect:CGRectMake(10, 55, 130, 80)];
    //图片添加图片水印
    //self.imageView.image = [self image:_imageView.image addMsakImage:[UIImage imageNamed:@"002.jpg"] msakRect:CGRectMake(0, 0, 130, 80)];
}
```
**2.实现刮刮乐效果**
```
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // 触摸任意位置
    UITouch *touch = touches.anyObject;
    // 触摸位置在图片上的坐标
    CGPoint cententPoint = [touch locationInView:self.imageView];
    // 设置清除点的大小
    CGRect  rect = CGRectMake(cententPoint.x, cententPoint.y, 30, 30);
    // 默认是去创建一个透明的视图
    UIGraphicsBeginImageContextWithOptions(self.imageView.bounds.size, NO, 0);
    // 获取上下文(画板)
    CGContextRef ref = UIGraphicsGetCurrentContext();
    // 把imageView的layer映射到上下文中
    [self.imageView.layer renderInContext:ref];
    // 清除划过的区域
    CGContextClearRect(ref, rect);
    // 获取图片
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    // 结束图片的画板, (意味着图片在上下文中消失)
    UIGraphicsEndImageContext();
    self.imageView.image = image;

}
```
**3.图片添加文字水印**
```
- (UIImage *)image:(UIImage *)image addText:(NSString *)mark msakRect:(CGRect)rect{
    int w = image.size.width;
    int h = image.size.height;
    UIGraphicsBeginImageContext(image.size);
    [[UIColor redColor] set];
    [image drawInRect:CGRectMake(0, 0, w, h)];
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];//改写字体属性
    dict[NSFontAttributeName] = [UIFont systemFontOfSize:30];//字号
    dict[NSForegroundColorAttributeName] = [UIColor grayColor];//颜色
    dict[NSStrokeWidthAttributeName] = @5;//空心
    [mark drawInRect:rect withAttributes:dict];
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}
```
![水印照片](/assets/blogImg/水印.png)
**4.图片添加图片水印**
```
- (UIImage *)image:(UIImage *)image addMsakImage:(UIImage *)maskImage msakRect:(CGRect)rect
{
    UIGraphicsBeginImageContext(image.size);
    [image drawInRect:CGRectMake(0, 0, image.size.width, image.size.height)];
    //四个参数为水印图片的位置
    [maskImage drawInRect:rect];
    UIImage *resultingImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return resultingImage;
}
```
![效果图](/assets/blogImg/刮刮乐.gif)



