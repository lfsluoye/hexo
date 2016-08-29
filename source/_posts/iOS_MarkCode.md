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
#### 2. 字典转json
最好写在分类里,一般上传都要用到
```
.h
+ (NSString *)dicToJson:(NSDictionary *)dic;
.m
+ (NSString *)dicToJson:(NSDictionary *)dic{
//dict转jsonData
NSData *jsonData = [NSJSONSerialization dataWithJSONObject: dic options:kNilOptions error:0];
//jsonData转String
NSString *jsonStr = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
return jsonStr;
}
```
#### 3. 日期字符串的降序排序
```
- (NSArray *)sortArray:(NSArray *)array{
NSStringCompareOptions comparisonOption = NSCaseInsensitiveSearch|NSNumericSearch|NSWidthInsensitiveSearch;
NSComparator sort = ^(NSString *obj1,NSString *obj2){
return [obj2 compare:obj1 options:comparisonOption];
};
return [array sortedArrayUsingComparator:sort];
}
```
#### 4. 查找同一级控件
```
- (void)uploadData:(UIButton *)btn{
for (UIView *subView in btn.superview.subviews) {
if ([subView isKindOfClass:[UILabel class]]) {
UILabel *label = (UILabel *)subView;
}
}
}
```


