---
layout: post
title: "Objective-C 与 Swift 混用"
date: 2015-06-08 
categories: iOS
comments: false
tags: OC
---
Swift 的学习已经提上日程，目前先在 Objective-C 的工程中试验，逐步重构。

入门自然先从官方文档和 WWDC 视频着手，[Mix Objective-C and Swift](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html#//apple_ref/doc/uid/TP40014216-CH10-XID_78) 给出了详实的内容，但却是一个糟糕的实践指导，而 WWDC 视频中也缺乏细节部分。
<!-- more -->
###桥接头文件
文档中指出，在同一个工程中在 OC 类中使用 Swift 类 或是从 Swift 类文件中使用 OC 类，都需要在一个头文件中为另一方导入类的接口，剩下的，只需在需要使用另一类别的类的类文件中引入该头文件即可。文档中分别给出了在 OC 中引入 Swift 和在 Swift 中引入 OC 两个主题，但当你实践的时候会发现细节上对不上，然后可能就出了什么问题不知所措。

文档先介绍了在 Swift 中引入 OC，但估计大部分尝试者应该是从在 OC 工程中使用 Swift 时开始，而不是并非全部 Swift 化。为什么文档要这么写？在 OC 工程中引入 Swift ，会发现还是会出现下面的过程，并自动生成了一个`ProductModuleName-Bridging-Header.h`的头文件。实际上，无论是在 Swift 工程中引入 OC 类还是在 OC 工程中引入 Swift 类，都会出现这个过程，这是在 Swift 中使用 OC 类的基础，由 Xcode 自动帮你完成，不会因为原来的工程代码是 Swift 还是 OC 而变化。
![swift中引入OC](/assets/blogImg/swift中引入OC.png)

如果没有生成这个文件，你需要手动建立这个文件。这个头文件用于向 Swift 类提供 OC 类的接口，如果你想要当前工程中的 OC 类能够在 Swift 类文件使用的话，在该头文件中引入需要的 OC 类即可，使用方式同普通的引入头文件没有区别，比如在该桥接头文件中加入`#import "XYZCustomView.h"`，在 Swift 类中引入该桥接头文件，那么就可以在 Swift 类中使用 XYZCustomView 类了。而想在 OC 中使用 Swift 类的话，则需要在 OC 类中引入"ProductModuleName-Swift.h"：`#import "ProductModuleName-Swift.h"`。这里有个很大的隐患，我刚开始没有意识到，`ProductModuleName-Swift.h`头文件由 Xcode 自动生成，而且你在工程中是看不到这个文件的，这是个大坑，不要自己手动建立这个文件，Xcode 会帮你处理好一切。在`ProductModuleName-Bridging-Header.h`中，你需要手动添加需要公开给 Swift 类使用的 OC 类，但 Swift 类并没有头文件，因此在 OC 类中使用 Swift 类的时候，你无法通过这样的方式来达到同样的效果，基于此，`ProductModuleName-Swift.h`头文件不可见（不可编辑，一切由 Xcode 来完成），Xcode 会根据 Swift 类的权限控制来限定公开给 OC 类的Swift 类。工程中的 Swift 类如果继承自 OC 类，则在默认设置下都可以在 OC 类中使用；如果不是继承自 OC 类，则需要添加 @objc 关键字使得该 Swift 类能够在 OC 类中使用。

总结下，在工程中，OC 类和 Swift 类使用对方类型的自定义类时，需要通过一个桥接头文件来访问对方；由于 OC 和 Swift 需要双向的互相访问，因此有两个头文件分别用于访问对方类型的类。这俩个头文件会由 Xcode 自动帮你生成，在 `ProductModuleName-Bridging-Header.h` 文件中指定需要公开给 Swift 类使用的 OC 类，`ProductModuleName-Swift.h `不可见，在 OC 类中引入该文件来使用 Swift 类，公开的 Swift 类范围则需要通过 Swift 类的权限控制来完成，相关官方文档：[Access Control](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html#//apple_ref/doc/uid/TP40014097-CH41-ID3)。具体操作如下：
![官方文档提供的使用条件](/assets/blogImg/官方提供的使用条件.png)

###Build Setting
仅仅完成上面的步骤会在后续的使用中会出现问题，需要进一步设置。这些设置都在 [Mix Objective-C and Swift](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html#//apple_ref/doc/uid/TP40014216-CH10-XID_78) 的「Troubleshooting Tips and Reminders」小节中指出来了，但不得不说把这部分内容放到最后会让很多人掉坑里，从这个帖子中可以看出来：[Can't use Swift classes inside Objective-C](http://stackoverflow.com/questions/24206732/cant-use-swift-classes-inside-objective-c)，该帖子对需要设置的内容做了总结：

> Open Build Settings and check this parameters:
Product Module Name : myproject
Defines Module : YES
Embedded Content Contains Swift : YES
Install Objective-C Compatibility Header : YES
Objective-C Bridging Header : $(SRCROOT)/Sources/SwiftBridging.h

在默认设置下，只要确保以下红框处的值为 YES 就可以了。

![Module Setting](/assets/blogImg/Module_Setting.png)
![Swift Module Setting](/assets/blogImg/Swift_Module_Setting.png)
###Product Module
两个头文件的前缀名称都是 Product Module Name，在默认情况下，它和工程的Product Name一样。
![ProductModuleSetting](/assets/blogImg/Product_Module_Setting.png)
你可以自定义该名称，Xcode 会根据该值来命名两个头文件。你不需要额外做什么，这里提到这个是因为你在 storyboard 中使用 Swift 类的时候多半会遇到这个问题：[Unknown Class in Interface Builder File](http://stackoverflow.com/questions/24924966/xcode-6-strange-bug-unknown-class-in-interface-builder-file)。解决方法很简单，为该 Swift 类指定 Module 即可，图片来自该贴下的回答。
![Unknown Class](/assets/blogImg/Unknown_Class.png)
由于糟糕的 Xcode，Module 的下拉选项到了 Xcode 6.3 中在很多时候无法提供有效值，这时候手动填写即可。