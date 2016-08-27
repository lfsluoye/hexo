---
layout: post
title: "Swift中使用随机数"
date: 2015-12-08 
categories: iOS
comments: false
tags: Swift
---
#### int类型的随机数
arc4random 是一个非常优秀的随机数算法,它会返回给我们一个任意的整数,如果我们想要在某一个范围里的话,做一次取模运算取余数就可以了.但是由于arc4random()函数返回的值无论上什么平台上都是返回一个UInt32(无符号32位整数)的值.**因此,在32位的平台上进行Int(arc4random())转换的话,就有一半的几率出现转换越界,这就会造成程序的崩溃.**
因此,在这种情况下,我们可以使用arc4random_uniform这个改良的arc4random函数:
<!-- more -->
```
func arc4random_uniform(_: UInt32) -> UInt32
```
这个函数接收一个UInt32的数字n作为输入,返回一个0到n-1之间的随机数.那么,只要我们传入的n不超过Int的范围,就可以避免像上面一样的转换越界的问题了.
因此,我们Int的随机数生成函数可以写成这样:
```
public extension Int {
    public static func random(lower: Int = 0, _ upper: Int = Int.max) -> Int {
        return lower + Int(arc4random_uniform(UInt32(upper - lower + 1)))
    }

    public static func random(range: Range<Int>) -> Int {
        return random(range.startIndex, range.endIndex)
    }
}
```
这段代码使用了扩展,对Int类型增加了一个扩展,并实现了random(lower: Int = 0, _ upper: Int = Int.max) -> Int和random(range: Range) -> Int两个方法.这样,如果我们需要一个整数的随机数的话,就可以这样调用了:
```
Int.random()
Int.random(0, 50)
Int.random(20...50)
```
这里不得不夸一下扩展机制,真心方便.它可以不修改原有类的源码的情况下,给这个类增加新的功能.这就大大的增加了编写代码的方便.毕竟很多时候最开始时是考虑不周全的,我们并不能给某个类增加所有的方法,到后面进行修改的时候,就必然牵涉到修改源码.如果源码是自己写的还好,如果源码不是自己写的或者根本就没有源码了.那么给这个类增加方法就非常的不方便了.

---

#### 其他类型的随机数
除了整数的随机数以外,浮点的随机数也是很常用的.因此我们同样可以对浮点数进行扩展.这里就有两种思路,第一种是继续使用arc4random()函数,把生成的随机整数转换成为浮点数.还有一种就是调用srand48(Int)和drand48()直接生成随机浮点数.这两个方案都是差不多的,不过由于每次调用drand48()前都需要调用srand48(Int)设置随机初始化的种子,因此我个人更倾向于使用arc4random().于是就有以下的方法:
```
public extension Bool {
    public static func random() -> Bool {
        return Int.random(0, 1) == 0
    }
}

public extension Double {
    /// SwiftRandom extension
    public static func random(lower: Double = 0, _ upper: Double = 100) -> Double {
        return (Double(arc4random()) / 0xFFFFFFFF) * (upper - lower) + lower
    }
}

public extension Float {
    /// SwiftRandom extension
    public static func random(lower: Float = 0, _ upper: Float = 100) -> Float {
        return (Float(arc4random()) / 0xFFFFFFFF) * (upper - lower) + lower
    }
}

public extension CGFloat {
    /// SwiftRandom extension
    public static func random(lower: CGFloat = 0, _ upper: CGFloat = 1) -> CGFloat {
        return CGFloat(Float(arc4random()) / Float(UINT32_MAX)) * (upper - lower) + lower
    }
}
```
这样集中常用数据类型的随机数的生成就都有了,并且使用起来也非常的方便.
```
Bool.random()
Double.random(1.5,10.8)
Float.random()
CGFloat.random()
```