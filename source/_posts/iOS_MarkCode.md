---
layout: post
title: "iOS不常用但是实用"
date: 2016-10-23
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
#### 9. ios通过正则判断6~18位包含数字或字母的字符串
```
- (BOOL)validateNumber:(NSString *) textString
{
NSString* number=@"^[0-9A-Za-z]{6,18}";
NSPredicate *numberPre = [NSPredicate predicateWithFormat:@"SELF MATCHES %@",number];
return [numberPre evaluateWithObject:textString];
}
```
#### 10. 替换同一个控件的文字大小或者颜色
```
let mutableStr = NSMutableAttributedString(string: monthlyRent)
let range = (monthlyRent as NSString).rangeOfString(kRentUnit)
mutableStr.addAttributes([NSFontAttributeName:UIFont.CustomSuitFont(12),NSForegroundColorAttributeName:KNewGreenTextHexColor], range: range)
label.attributedText = mutableStr
```
#### 11. 延迟
```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (Int64)(1 * NSEC_PER_SEC)), dispatch_get_main_queue()) { [weak self]() -> Void in
}
```
#### 12. 完成编辑
```
func scrollViewWillBeginDragging(scrollView: UIScrollView) {
self.view.endEditing(true)
}
```

#### 13. 渲染图片颜色
```
/**
该图片染色

- parameter color: 要染的颜色

- returns: 返回染完的图片
*/
func imageTinted(color:UIColor) -> UIImage{
UIGraphicsBeginImageContextWithOptions(self.size, false, 0.0);
var rect = CGRectZero
rect.size = self.size
color.set()
UIRectFill(rect)
drawInRect(rect, blendMode: CGBlendMode.DestinationIn, alpha: 1.0)
let image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
return image
}
```

#### 14. 将view转为图片
```
//    将某个view 转换成图像
class func getImageFromView(view:UIView) ->UIImage{
UIGraphicsBeginImageContext(view.bounds.size)
view.layer.render(in: UIGraphicsGetCurrentContext()!)

let image = UIGraphicsGetImageFromCurrentImageContext()

UIGraphicsEndImageContext()

return image!
}
// 给定指定宽度，返回结果图像
func scaleImageToWidth(_ width: CGFloat) -> UIImage {

// 1. 判断宽度，如果小于指定宽度直接返回当前图像

if size.width < width {

return self

}

// 2. 计算等比例缩放的高度

let height = width * size.height / size.width

// 3. 图像的上下文

let s = CGSize(width: width, height: height)

// 提示：一旦开启上下文，所有的绘图都在当前上下文中

UIGraphicsBeginImageContext(s)

// 在制定区域中缩放绘制完整图像

draw(in: CGRect(origin: CGPoint.zero, size: s))

// 4. 获取绘制结果

let result = UIGraphicsGetImageFromCurrentImageContext()

// 5. 关闭上下文

UIGraphicsEndImageContext()

// 6. 返回结果

return result!

}
```


