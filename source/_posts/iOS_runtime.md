---
layout: post
title: "iOS--runtime"
date: 2016-01-21 
categories: iOS
comments: false
tags: OC
---

## iOS runtime理论篇
### 前言
终于抽出时间整理属于自己的runtime了
先上图
![runtime学习图](/assets/blogImg/runtime学习图.png)
### runtime是什么
对于初学者，runtime如尼斯湖水怪一样，只存在于传说中，对于开发者，runtime是做好iOS开发，或是深刻掌握Objective C所必需理解的东西。大公司面试都喜欢问：你对runtime熟悉吗？并不是runtime在开发中经常用到，我认为它是OC最核心的部分，只有掌握好它，你才能理解其底层的原理，而不是做一个只会造轮子的码农。
<!-- more -->
#### 1. 那么runtime到底是什么鬼？
> runtime是一个c和汇编写的动态库，它就像一个小小的系统，将OC和C紧密关联，这个系统主要做两件事 ：
1、封装C语言的结构体和函数，让开发者在运行时创建、检查或者修改类、对象和方法等等。
2、传递消息，找出方法的最终执行代码。

听起来蛮抽象的，我们来点通俗的吧？没问题～～
我们先写一句OC的代码
> [zhangsan walkTheDog];

那么在运行时runtime会将它转化成C语言的代码
> objc_msgSend(zhangsan, @selector(walkTheDog));

这个方法就是发送消息的方法，类似这样的方法runtime提供了很多，比如：
> objc_property_t * class_copyPropertyList ( Class cls, unsigned int *outCount ); // 获取属性列表
Method * class_copyMethodList ( Class cls, unsigned int *outCount );            // 获取所有方法的数组
BOOL class_addMethod ( Class cls, SEL name, IMP imp, const char *types );       // 添加方法


那么我们可以利用这些方法干点什么？
> 1、遍历对象的属性
比如，看看zhangsan的有哪些属性（身高：180、年龄：18）

> 2、动态添加/修改属性，动态添加/修改/替换方法
比如，修改zhangsan的身高为190、年龄为20，替换walkTheDog方法（变成walkTheBigDog），给他添加一个新方法（walkTheCat）等等

> 3、动态创建类/对象/协议等等
比如，创建一个新的对象：lisi

> 4、方法拦截调用
比如，给zhangsan发送一个walkTheDog消息，但是zhangsan不知道怎么walk啊（没实现该方法），那我们可以拦截下，给该方法动态添加一个实现，甚至可以讲该方法定向或者打包给lisi（其他对象），让lisi来walk。

#### 2. 方法调用流程
通俗地讲，调用方法（包含实例方法和类方法）相当于給一个对象发送消息。
> 所以，实际上，类本身也是一个对象（关于Class这一块就不再这里展开了）。
当我们调用一个方法时，是这样的：
Instance：调用实例方法时，会到对象所属的类的方法列表中查找。
Class：调用类方法时，会到类的metaClass的方法列表中查找。

下面以实例对象调用方法[blackDog walk]为例描述方法调用的流程：
> 1、编译器会把`[blackDog walk]`转化为`objc_msgSend(blackDog，SEL)`，SEL为@selector(walk)。

> 2、Runtime会在blackDog对象所对应的Dog类的方法缓存列表里查找方法的SEL

> 3、如果没有找到，则在Dog类的方法分发表查找方法的SEL。（类由对象isa指针指向，方法分发表即methodList）

> 4、如果没有找到，则在其父类（设Dog类的父类为Animal类）的方法分发表里查找方法的SEL（父类由类的superClass指向）

> 5、如果没有找到，则沿继承体系继续下去，最终到达NSObject类。

> 6、如果在234的其中一步中找到，则定位了方法实现的入口，执行具体实现

> 7、如果最后还是没有找到，会面临两种情况：
(1) 如果是使用［blackDog walk］的方式调用方法
(2) 使用［blackDog performSelector:@selector(walk)］的方式调用方法`
第一种情况编译器会报错，第二种需要到运行时才能确定对象能否接收指定的消息，这时候会进入消息转发的流程

#### 3. 消息转发流程
> 1. 动态方法解析
接收到未知消息时（假设blackDog的walk方法尚未实现），runtime会调用+resolveInstanceMethod:（实例方法）或者+resolveClassMethod:（类方法）

> 2. 备用接收者
如果以上方法没有做处理，runtime会调用- (id)forwardingTargetForSelector:(SEL)aSelector方法。
如果该方法返回了一个非nil（也不能是self）的对象，而且该对象实现了这个方法，那么这个对象就成了消息的接收者，消息就被分发到该对象。
适用情况：通常在对象内部使用，让内部的另外一个对象处理消息，在外面看起来就像是该对象处理了消息。
比如：blackDog让女朋友whiteDog来接收这个消息

> 3. 完整消息转发
在- (void)forwardInvocation:(NSInvocation *)anInvocation方法中选择转发消息的对象，其中anInvocation对象封装了未知消息的所有细节，并保留调用结果发送到原始调用者。
比如：blackDog将消息完整转发給主人dogOwner来处理

![消息转发流程图](/assets/blogImg/消息转发流程图.png)

### 成员变量和属性
曾遇到这样一个问题：“你知道成员变量的本质是什么吗？”
立马懵逼了，成员变量的本质？成员变量就是成员变量啊，平时只管用，还有什么更深层的含义？本文着重介绍runtime中成员变量和属性的定义和使用。
#### 1. 成员变量
##### 1.1 定义：
Ivar: 实例变量类型，是一个指向objc_ivar结构体的指针
> typedef struct objc_ivar *Ivar;

##### 1.2 操作函数：
> // 获取所有成员变量
class_copyIvarList

> // 获取成员变量名
ivar_getName

> // 获取成员变量类型编码
ivar_getTypeEncoding

> // 获取指定名称的成员变量
class_getInstanceVariable

> // 获取某个对象成员变量的值
object_getIvar

> // 设置某个对象成员变量的值
object_setIvar

##### 1.3 使用实例：
Model的头文件声明如下：
```
@interface Model : NSObject {
        NSString * _str1;
    }
    @property NSString * str2;
    @property (nonatomic, copy) NSDictionary * dict1;
    @end
```
获取其成员变量
```
unsigned int outCount = 0;
    Ivar * ivars = class_copyIvarList([Model class], &outCount);
    for (unsigned int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        const char * name = ivar_getName(ivar);
        const char * type = ivar_getTypeEncoding(ivar);
        NSLog(@"类型为 %s 的 %s ",type, name);
    }
    free(ivars);
```
打印结果：
```
runtimeIvar[602:16885] 类型为 @"NSString" 的 _str1 
runtimeIvar[602:16885] 类型为 @"NSString" 的 _str2 
runtimeIvar[602:16885] 类型为 @"NSDictionary" 的 _dict1
```

#### 2. 属性
##### 2.1 定义：
objc_property_t：声明的属性的类型，是一个指向objc_property结构体的指针
> typedef struct objc_property *objc_property_t;

##### 2.2 操作函数
> // 获取所有属性
class_copyPropertyList
说明：使用class_copyPropertyList并不会获取无@property声明的成员变量

> // 获取属性名
property_getName

> // 获取属性特性描述字符串
property_getAttributes

> // 获取所有属性特性
property_copyAttributeList

>说明：
property_getAttributes函数返回objc_property_attribute_t结构体列表，objc_property_attribute_t结构体包含name和value，常用的属性如下：

> 属性类型  name值：T  value：变化
编码类型  name值：C(copy) &(strong) W(weak) 空(assign) 等 value：无
非/原子性 name值：空(atomic) N(Nonatomic)  value：无
变量名称  name值：V  value：变化

使用property_getAttributes获得的描述是property_copyAttributeList能获取到的所有的name和value的总体描述，如 T@"NSDictionary",C,N,V_dict1

##### 2.3 使用实例：
```
unsigned int outCount = 0;
    objc_property_t * properties = class_copyPropertyList([Model class], &outCount);
    for (unsigned int i = 0; i < outCount; i ++) {
        objc_property_t property = properties[i];
        //属性名
        const char * name = property_getName(property);
        //属性描述
        const char * propertyAttr = property_getAttributes(property);
        NSLog(@"属性描述为 %s 的 %s ", propertyAttr, name);

        //属性的特性
        unsigned int attrCount = 0;
        objc_property_attribute_t * attrs = property_copyAttributeList(property, &attrCount);
        for (unsigned int j = 0; j < attrCount; j ++) {
            objc_property_attribute_t attr = attrs[j];
            const char * name = attr.name;
            const char * value = attr.value;
            NSLog(@"属性的描述：%s 值：%s", name, value);
        }
        free(attrs);
        NSLog(@"\n");
    }
    free(properties);
```
打印结果：
```
runtimeIvar[661:27041] 属性描述为 T@"NSString",&,V_str2 的 str2 
runtimeIvar[661:27041] 属性的描述：T 值：@"NSString"
runtimeIvar[661:27041] 属性的描述：& 值：
runtimeIvar[661:27041] 属性的描述：V 值：_str2
runtimeIvar[661:27041] 
runtimeIvar[661:27041] 属性描述为 T@"NSDictionary",C,N,V_dict1 的 dict1 
runtimeIvar[661:27041] 属性的描述：T 值：@"NSDictionary"
runtimeIvar[661:27041] 属性的描述：C 值：
runtimeIvar[661:27041] 属性的描述：N 值：
runtimeIvar[661:27041] 属性的描述：V 值：_dict1
runtimeIvar[661:27041]
```

## runtime实战篇
### 1. Json到Model的转化

在开发中相信最常用的就是接口数据需要转化成Model了（当然如果你是直接从Dict取值的话。。。），很多开发者也都使用著名的第三方库如JsonModel、Mantle或MJExtension等，如果只用而不知其所以然，那真和“搬砖”没啥区别了，下面我们使用runtime去解析json来给Model赋值。

原理描述：用runtime提供的函数遍历Model自身所有属性，如果属性在json中有对应的值，则将其赋值。
核心方法：在NSObject的分类中添加方法：
```
- (instancetype)initWithDict:(NSDictionary *)dict {

    if (self = [self init]) {
        //(1)获取类的属性及属性对应的类型
        NSMutableArray * keys = [NSMutableArray array];
        NSMutableArray * attributes = [NSMutableArray array];
        /*
         * 例子
         * name = value3 attribute = T@"NSString",C,N,V_value3
         * name = value4 attribute = T^i,N,V_value4
         */
        unsigned int outCount;
        objc_property_t * properties = class_copyPropertyList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            objc_property_t property = properties[i];
            //通过property_getName函数获得属性的名字
            NSString * propertyName = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            [keys addObject:propertyName];
            //通过property_getAttributes函数可以获得属性的名字和@encode编码
            NSString * propertyAttribute = [NSString stringWithCString:property_getAttributes(property) encoding:NSUTF8StringEncoding];
            [attributes addObject:propertyAttribute];
        }
        //立即释放properties指向的内存
        free(properties);

        //(2)根据类型给属性赋值
        for (NSString * key in keys) {
            if ([dict valueForKey:key] == nil) continue;
            [self setValue:[dict valueForKey:key] forKey:key];
        }
    }
    return self;

}
```

读者可以进一步思考：
1、如何识别基本数据类型的属性并处理
2、空（nil，null）值的处理
3、json中嵌套json（Dict或Array）的处理
尝试解决以上问题，你也能写出属于自己的功能完备的Json转Model库。

### 2. 快速归档
有时候我们要对一些信息进行归档，如用户信息类UserInfo，这将需要重写initWithCoder和encodeWithCoder方法，并对每个属性进行encode和decode操作。那么问题来了：当属性只有几个的时候可以轻松写完，如果有几十个属性呢？那不得写到天荒地老？。。。

原理描述：用runtime提供的函数遍历Model自身所有属性，并对属性进行encode和decode操作。
核心方法：在Model的基类中重写方法：
```
- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int outCount;
        Ivar * ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar ivar = ivars[i];
            NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            [self setValue:[aDecoder decodeObjectForKey:key] forKey:key];
        }
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
        [aCoder encodeObject:[self valueForKey:key] forKey:key];
    }
}
```

### 3. 访问私有变量
我们知道，OC中没有真正意义上的私有变量和方法，要让成员变量私有，要放在m文件中声明，不对外暴露。如果我们知道这个成员变量的名称，可以通过runtime获取成员变量，再通过getIvar来获取它的值。
方法：
>     Ivar ivar = class_getInstanceVariable([Model class], "_str1");
    NSString * str1 = object_getIvar(model, ivar);
    
### 4. 关联对象
##### 4.1 怎样关联对象
runtime提供給我们的方法：
```
//关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
//获取关联的对象
id objc_getAssociatedObject(id object, const void *key)
//移除关联的对象
void objc_removeAssociatedObjects(id object)
```
变量说明
> id object：被关联的对象（如xiaoming）
const void *key：关联的key，要求唯一
id value：关联的对象（如dog）
objc_AssociationPolicy policy：内存管理的策略

objc_AssociationPolicy policy的enum值有
> OBJC_ASSOCIATION_ASSIGN = 0,          
OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, 
OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  
OBJC_ASSOCIATION_RETAIN = 01401,       
OBJC_ASSOCIATION_COPY = 01403

当对象被释放时，会根据这个策略来决定是否释放关联的对象，当策略是RETAIN/COPY时，会释放（release）关联的对象，当是ASSIGN，将不会释放。
值得注意的是，我们不需要主动调用removeAssociated来接触关联的对象，如果需要解除指定的对象，可以使用setAssociatedObject置nil来实现。
##### 4.2 关联对象的应用
##### 1. 添加公共属性
这是最常用的一个模式，通常我们会在类声明里面添加属性，但是出于某些需求（如前言描述的情况），我们需要在分类里添加一个或多个属性的话，编译器就会报错，这个问题的解决方案就是使用runtime的关联对象。
应用举例：
我们需要自定义一个tabbar，并暴露公共的属性和方法。（读者们可以思考下使用继承和分类实现的优点和不足之处）
```
@interface UITabBarController (Custom)

@property (nonatomic, strong) SUCustomTabbar * customTabbar;

@end
```
```
#import "UITabBarController+Custom.h"
#import <objc/runtime.h>

@implementation UITabBarController (Custom)

- (void)setCustomTabbar:(UIView *)customTabbar {
    //这里使用方法的指针地址作为唯一的key
    objc_setAssociatedObject(self, @selector(customTabbar), customTabbar, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (UIView *)customTabbar {
    return objc_getAssociatedObject(self, @selector(customTabbar));
}

//其他方法...

@end
```

这样，我们就可以像原生的tabbar一样使用自定义的tabbar：

> [self.tabBarController.customTabbar doSomgthig];

##### 2. 添加私有成员变量
有时候，需要在分类中添加不想暴露在公共声明的成员变量。
应用举例：給按钮添加点击事件的回调
_.h_
```
@interface UIButton (Callback)

- (instancetype)initWithFrame:(CGRect)frame callback:(void (^)(UIButton *))callbackBlock;

@end
```
_.m_
```
@interface UIButton ()

@property (nonatomic, copy) void (^callbackBlock)(UIButton * button);

@end

@implementation UIButton (Callback)

- (void (^)(UIButton *))callbackBlock {
    return objc_getAssociatedObject(self, @selector(callbackBlock));
}

- (void)setCallbackBlock:(void (^)(UIButton *))callbackBlock {
    objc_setAssociatedObject(self, @selector(callbackBlock), callbackBlock, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (instancetype)initWithFrame:(CGRect)frame callback:(void (^)(UIButton *))callbackBlock {

    if (self = [super initWithFrame:frame]) {
        self.callbackBlock = callbackBlock;
        [self addTarget:self action:@selector(didClickAction:) forControlEvents:UIControlEventTouchUpInside];
    }
    return self;
}

- (void)didClickAction:(UIButton *)button {
    self.callbackBlock(button);
}

@end
```

### 5. Method Swizzling
> method Swizzling原理
每个类都维护一个方法（Method）列表，Method则包含SEL和其对应IMP的信息，方法交换做的事情就是把SEL和IMP的对应关系断开，并和新的IMP生成对应关系
```
+ (void)load {

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        Class selfClass = object_getClass([self class]);

        SEL oriSEL = @selector(imageNamed:);
        Method oriMethod = class_getInstanceMethod(selfClass, oriSEL);

        SEL cusSEL = @selector(myImageNamed:);
        Method cusMethod = class_getInstanceMethod(selfClass, cusSEL);

        BOOL addSucc = class_addMethod(selfClass, oriSEL, method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));
        if (addSucc) {
            class_replaceMethod(selfClass, cusSEL, method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
        }else {
            method_exchangeImplementations(oriMethod, cusMethod);
        }

    });
}
```

另一种
```
#import

@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```
简要说明一下以上代码的几个重点:

- 通过在Category的+ (void)load方法中添加Method Swizzling的代码,在类初始加载时自动被调用,load方法按照父类到子类,类自身到Category的顺序被调用.
- 在dispatch_once中执行Method Swizzling是一种防护措施,以保证代码块只会被执行一次并且线程安全,不过此处并不需要,因为当前Category中的load方法并不会被多次调用.
- 尝试先调用class_addMethod方法,以保证即便originalSelector只在父类中实现,也能达到Method Swizzling的目的.
- xxx_viewWillAppear:方法中[self xxx_viewWillAppear:animated];代码并不会造成死循环,因为Method Swizzling之后, 调用xxx_viewWillAppear:实际执行的代码已经是原来viewWillAppear中的代码了.
其实以上的代码也可以简写为以下:
```
+ (void)load {
    Class class = [self class];
    
    SEL originalSelector = @selector(viewWillAppear:);
    SEL swizzledSelector = @selector(xxx_viewWillAppear:);
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    if (!originalMethod || !swizzledMethod) {
        return;
    }
    
    IMP originalIMP = method_getImplementation(originalMethod);
    IMP swizzledIMP = method_getImplementation(swizzledMethod);
    const char *originalType = method_getTypeEncoding(originalMethod);
    const char *swizzledType = method_getTypeEncoding(swizzledMethod);
    
    class_replaceMethod(class,originalSelector,swizzledIMP,swizzledType);
    class_replaceMethod(class,swizzledSelector,originalIMP,originalType);
}
```
这是因为class_replaceMethod方法其实能够覆盖到class_addMethod和method_setImplementation两种场景, 对于第一个class_replaceMethod来说, 如果viewWillAppear:实现在父类, 则执行class_addMethod, 否则就执行method_setImplementation将原方法的IMP指定新的代码块; 而第二个class_replaceMethod完成的工作便只是将新方法的IMP指向原来的代码.

除了以上的场景之外,其它场景下我们如何使用Method Swizzling呢?

##### 1. 在不同类之间实现Method Swizzling

上面示例是通过Category来新增一个方法然后实现Method Swizzling的, 但有一些场景可能并不适合使用Category(比如私有的类,未获取到该类的声明), 此时我们应该如何来做Method Swizzling呢?

例如已知一个className为Car的类中有一个实例方法- (void)run:(double)speed, 目前需要Hook该方法对速度小于120才执行run的代码, 按照方法交换的流程, 代码应该是这样的:
```
#import 

@interface MyCar : NSObject
@end

@implementation MyCar

+ (void)load {
    Class originalClass = NSClassFromString(@"Car");
    Class swizzledClass = [self class];
    SEL originalSelector = NSSelectorFromString(@"run:");
    SEL swizzledSelector = @selector(xxx_run:);
    Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
    
    // 向Car类中新添加一个xxx_run:的方法
    BOOL registerMethod = class_addMethod(originalClass,
                                          swizzledSelector,
                                          method_getImplementation(swizzledMethod),
                                          method_getTypeEncoding(swizzledMethod));
    if (!registerMethod) {
        return;
    }
    
    // 需要更新swizzledMethod变量,获取当前Car类中xxx_run:的Method指针
    swizzledMethod = class_getInstanceMethod(originalClass, swizzledSelector);
    if (!swizzledMethod) {
        return;
    }
    
    // 后续流程与之前的一致
    BOOL didAddMethod = class_addMethod(originalClass,
                                        originalSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {
        class_replaceMethod(originalClass,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

- (void)xxx_run:(double)speed {
    if (speed < 120) {
        [self xxx_run:speed];
    }
}

@end
```
与之前的流程相比,在前面添加了两个逻辑:

利用runtime向目标类Car动态添加了一个新的方法,此时Car类与MyCar类一样具备了xxx_run:这个方法,MyCar的利用价值便结束了;
为了完成后续Car类中run:与xxx_run:的方法交换,此时需要更新swizzledMethod变量为Car中的xxx_run:方法所对应的Method.
以上所有的逻辑也可以合并简化为以下:
```
+ (void)load {
    Class originalClass = NSClassFromString(@"Car");
    Class swizzledClass = [self class];
    SEL originalSelector = NSSelectorFromString(@"run:");
    SEL swizzledSelector = @selector(xxx_run:);
    Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
    
    IMP originalIMP = method_getImplementation(originalMethod);
    IMP swizzledIMP = method_getImplementation(swizzledMethod);
    const char *originalType = method_getTypeEncoding(originalMethod);
    const char *swizzledType = method_getTypeEncoding(swizzledMethod);
    
    class_replaceMethod(originalClass,swizzledSelector,originalIMP,originalType);
    class_replaceMethod(originalClass,originalSelector,swizzledIMP,swizzledType);
}
```
简化后的代码便与之前使用Category的方式并没有什么差异, 这样代码就很容易覆盖到这两种场景了, 但我们需要明确此时class_replaceMethod所完成的工作却是不一样的.

>- 第一个class_replaceMethod与之前的逻辑一致, 当run:方法是实现在Car类或Car的父类, 分别执行method_setImplementation或class_addMethod;
- 第二个class_replaceMethod则直接在Car类中注册了xxx_run:方法, 并且指定的IMP为当前run:方法的IMP;

2. 如何实现类方法的Method Swizzling

以上的代码都是实现的对实例方法的交换, 那如何来实现对类方法的交换呢, 依旧直接贴代码吧:
```
@interface NSDictionary (Test)
@end

@implementation NSDictionary (Test)

+ (void)load {
    Class cls = [self class];
    SEL originalSelector = @selector(dictionary);
    SEL swizzledSelector = @selector(xxx_dictionary);
    
    // 使用class_getClassMethod来获取类方法的Method
    Method originalMethod = class_getClassMethod(cls, originalSelector);
    Method swizzledMethod = class_getClassMethod(cls, swizzledSelector);
    if (!originalMethod || !swizzledMethod) {
        return;
    }
    
    IMP originalIMP = method_getImplementation(originalMethod);
    IMP swizzledIMP = method_getImplementation(swizzledMethod);
    const char *originalType = method_getTypeEncoding(originalMethod);
    const char *swizzledType = method_getTypeEncoding(swizzledMethod);
    
    // 类方法添加,需要将方法添加到MetaClass中
    Class metaClass = objc_getMetaClass(class_getName(cls));
    class_replaceMethod(metaClass,originalSelector,swizzledIMP,swizzledType);
    class_replaceMethod(metaClass,swizzledSelector,originalIMP,originalType);
}

+ (id)xxx_dictionary {
    id result = [self xxx_dictionary];
    return result;
}

@end
```
相比实例方法的Method Swizzling,流程有两点差异:
>- 获取Method的方法变更为class_getClassMethod(Class cls, SEL name),从函数命名便直观体现了和class_getInstanceMethod(Class cls, SEL name)的差别;
- 对于类方法的动态添加,需要将方法添加到MetaClass中,因为实例方法记录在class的method-list中, 类方法是记录在meta-class中的method-list中的.

##### 3. 在类簇中如何实现Method Swizzling
在上面的代码中我们实现了对NSDictionary中的+ (id)dictionary方法的交换,但如果我们用类似代码尝试对- (id)objectForKey:(id)key方法进行交换后, 你便会发现这似乎并没有什么用.

这是为什么呢? 平常我们在Xcode调试时,在下方Debug区域左侧的Variables View中,常常会发现如__NSArrayI或是__NSCFConstantString这样的Class类型, 这便是在Foundation框架中被广泛使用的类簇, 详情请参看Apple文档class cluster的内容.

所以针对类簇的Method Swizzling问题就转变为如何对这些类簇中的私有类做Method Swizzling, 在上面介绍的不同类之间做Method Swizzling便已经能解决该问题, 下面一个简单的示例通过交换NSMutableDictionary的setObject:forKey:方法,让调用这个方法时当参数object或key为空的不会抛出异常:
```
@interface MySafeDictionary : NSObject
@end

@implementation MySafeDictionary

+ (void)load {
    Class originalClass = NSClassFromString(@"__NSDictionaryM");
    Class swizzledClass = [self class];
    SEL originalSelector = @selector(setObject:forKey:);
    SEL swizzledSelector = @selector(safe_setObject:forKey:);
    Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
    
    IMP originalIMP = method_getImplementation(originalMethod);
    IMP swizzledIMP = method_getImplementation(swizzledMethod);
    const char *originalType = method_getTypeEncoding(originalMethod);
    const char *swizzledType = method_getTypeEncoding(swizzledMethod);
    
    class_replaceMethod(originalClass,swizzledSelector,originalIMP,originalType);
    class_replaceMethod(originalClass,originalSelector,swizzledIMP,swizzledType);
}

- (void)safe_setObject:(id)anObject forKey:(id)aKey {
    if (anObject && aKey) {
        [self safe_setObject:anObject forKey:aKey];
    }
    else if (aKey) {
        [(NSMutableDictionary *)self removeObjectForKey:aKey];
    }
}

@end
```
##### 4. 在Method Swizzling之后如何恢复
使用了Method Swizzling的各种姿势之后, 是否有考虑如何恢复到交换之前的现场呢?

一种方案就是通过一个开关标识符, 如果需要从逻辑上面恢复到交换之前, 就设置一下这个标识符, 在实现中判定如果设定了该标识符, 逻辑就直接调用原方法的实现, 其它什么事儿也不干, 这是目前大多数代码的实现方法, 当然也是非常安全的方式, 只不过当交换方法过多时, 每一个交换的方法体中都需要增加这样的逻辑, 并且也需要维护大量这些标识符变量, 只是会觉得不够优雅, 所以此处也就不展开详细讨论了.

那下面来讨论一下有没有更好的方案, 以上描述的Method Swizzling各种场景和处理的技巧, 但综合总结之后最核心的其实也只做了两件事情:

>- class_addMethod 添加一个新的方法, 可能是把其它类中实现的方法添加到目标类中, 也可能是把父类实现的方法添加一份在子类中, 可能是添加的实例方法, 也可能是添加的类方法, 总之就是添加了方法.
- 交换IMP 交换方法的实现IMP,完成这个步骤除了使用method_exchangeImplementations这个方法外, 也可以是调用了method_setImplementation方法来单独修改某个方法的IMP, 或者是采用在调用class_addMethod方法中设定了IMP而直接就完成了IMP的交换, 总之就是对IMP的交换.

那我们来分别看一下这两件事情是否都还能恢复:

>- 对于class_addMethod, 我们首先想到的可能就是有没有对应的remove方法呢, 在Objective-C 1.0的时候有class_removeMethods这个方法, 不过在2.0的时候就已经被禁用了, 也就是苹果并不推荐我们这样做, 想想似乎也是挺有道理的, 本来runtime的接口看着就挺让人心惊胆战的, 又是添加又是删除总觉得会出岔子, 所以只能放弃remove的想法, 反正方法添加在那儿倒也没什么太大的影响.
- 针对IMP的交换, 在Method Swizzling时做的交换动作, 如果需要恢复其实要做的动作还是交换回来罢了, 所以是可以做到的, 不过需要怎样做呢? 对于同一个类, 同一个方法, 可能会在不同的地方被多次做Method Swizzling, 所以要回退某一次的Method Swizzling, 我们就需要记录下来这一次交换的时候是哪两个IMP做了交换, 恢复的时候再换回来即可. 另一个问题是如果已经经过多次交换, 我们怎样找到这两个IMP所对应的Mehod呢, 还好runtime提供了一个class_copyMethodList方法, 可以直接取出Method列表, 然后我们就可以逐个遍历找到IMP所对应的Method了, 下面是对上一个示例添加恢复之后实现的代码逻辑:
```
#import 

@interface MySafeDictionary : NSObject
@end

static NSLock *kMySafeLock = nil;
static IMP kMySafeOriginalIMP = NULL;
static IMP kMySafeSwizzledIMP = NULL;

@implementation MySafeDictionary

+ (void)swizzlling {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        kMySafeLock = [[NSLock alloc] init];
    });
    
    [kMySafeLock lock];
    
    do {
        if (kMySafeOriginalIMP || kMySafeSwizzledIMP) break;
        
        Class originalClass = NSClassFromString(@"__NSDictionaryM");
        if (!originalClass) break;
        
        Class swizzledClass = [self class];
        SEL originalSelector = @selector(setObject:forKey:);
        SEL swizzledSelector = @selector(safe_setObject:forKey:);
        Method originalMethod = class_getInstanceMethod(originalClass, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(swizzledClass, swizzledSelector);
        if (!originalMethod || !swizzledMethod) break;
        
        IMP originalIMP = method_getImplementation(originalMethod);
        IMP swizzledIMP = method_getImplementation(swizzledMethod);
        const char *originalType = method_getTypeEncoding(originalMethod);
        const char *swizzledType = method_getTypeEncoding(swizzledMethod);
        
        kMySafeOriginalIMP = originalIMP;
        kMySafeSwizzledIMP = swizzledIMP;
        
        class_replaceMethod(originalClass,swizzledSelector,originalIMP,originalType);
        class_replaceMethod(originalClass,originalSelector,swizzledIMP,swizzledType);
    } while (NO);
    
    [kMySafeLock unlock];
}

+ (void)restore {
    [kMySafeLock lock];
    
    do {
        if (!kMySafeOriginalIMP || !kMySafeSwizzledIMP) break;
        
        Class originalClass = NSClassFromString(@"__NSDictionaryM");
        if (!originalClass) break;
        
        unsigned int outCount = 0;
        Method *methodList = class_copyMethodList(originalClass, &outCount);
        for (unsigned int idx=0; idx < outCount; idx++) {
            Method aMethod = methodList[idx];
            IMP aIMP = method_getImplementation(aMethod);
            if (aIMP == kMySafeSwizzledIMP) {
                method_setImplementation(aMethod, kMySafeOriginalIMP);
            }
            else if (aIMP == kMySafeOriginalIMP) {
                method_setImplementation(aMethod, kMySafeSwizzledIMP);
            }
        }
        kMySafeOriginalIMP = NULL;
        kMySafeSwizzledIMP = NULL;
    } while (NO);
    
    [kMySafeLock unlock];
}

- (void)safe_setObject:(id)anObject forKey:(id)aKey {
    if (anObject && aKey) {
        [self safe_setObject:anObject forKey:aKey];
    }
    else if (aKey) {
        [(NSMutableDictionary *)self removeObjectForKey:aKey];
    }
}

@end
```
注意 这段代码的Method Swizzling和恢复都需要主动调用, 并且相比上面其它的示例, 这段代码还添加如锁机制来加之保护. 这个示例是以不同的类来实现的Method Swizzling和恢复, 如果是Category或者是类方法, 根据之前的示例也需要做相应的调整.