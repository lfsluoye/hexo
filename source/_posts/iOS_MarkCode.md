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
<!-- more -->
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

#### 5. 设备唯一标识
```
+ (NSString *)udid {
    NSString * desStr = [NSString stringWithFormat:@"%@",[[UIDevice currentDevice] identifierForVendor].UUIDString];
    
    return desStr;
}
```
#### 6. 判断字符串是否有中文
```
+(BOOL)IsChinese:(NSString *)str {
    for(int i=0; i< [str length];i++){
        int a = [str characterAtIndex:i];
        if( a > 0x4e00 && a < 0x9fff){
            return YES;
        }
    } return NO;
}
```

#### 7. 获得随机码uuid
```
- (NSString *)uuid {
    CFUUIDRef puuid = CFUUIDCreate(nil);
    CFStringRef uuidString = CFUUIDCreateString(nil, puuid);
    NSString * result = (NSString *)CFBridgingRelease(CFStringCreateCopy(NULL, uuidString));
    CFRelease(puuid);
    CFRelease(uuidString);
    return result.lowercaseString;
}
+ (NSString *)uuid {
    return [[[self alloc] init] uuid];
}
```

#### 8. MD5 32位加密
```
- (NSString *)md532BitUpper:(NSString *)str {
    const char * cStr = [str UTF8String];
    unsigned char result[16];
    NSNumber * num = [NSNumber numberWithUnsignedLong:strlen(cStr)];
    CC_MD5(cStr, [num intValue], result);

    NSMutableString * ret = [NSMutableString stringWithCapacity:CC_MD5_DIGEST_LENGTH];
    for (long i = 0; i < CC_MD5_DIGEST_LENGTH; i++) {
        [ret appendFormat:@"%02x",result[i]];
    }
    return ret;
}

+ (NSString *)md532Bitupper:(NSString *)str {
    return [[[self alloc] init] md532BitUpper:str];
}
```
