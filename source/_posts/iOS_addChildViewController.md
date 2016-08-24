---
layout: post
title: "iOS addChildViewController方法"
date: 2016-8-23 
categories: iOS
comments: false
tags: OC 
---

APP中经常有根据标签来切换页面的需求，如果切换的页面只是刷新一下数据也就罢了，但是如果每个标签切换页面的数据和内容、结构完全不同你会怎么样做？
**个人觉得理想的做法就是每个标签展示的内容为一个View，这样切换既不会影响之前View还可以快速切回之前的View，而且符合高聚合、低耦合开发啊，这里就要隆重介绍一下addChildViewController方法：**
<!-- more -->
  
```OC
//在ViewController 中添加其他UIViewController，currentVC是一个UIViewController变量，存储当前显示的viewcontroller
    FirstVC * first = [[FirstVC alloc] init];
    [self addChildViewController:first];
    //addChildViewController 会调用 [child willMoveToParentViewController:self] 方法，但是不会调用 didMoveToParentViewController:方法，官方建议显示调用
    [self didMoveToParentViewController:first];
    [first.view setFrame:CGRectMake(0, CGRectGetMaxY(myScrollView.frame), width, height-CGRectGetHeight(myScrollView.frame))];
    currentVC = first;
    [self.view addSubview:currentVC.view];
//这里没有其他addSubview:方法了，就只有一个，而且可以切换视图，是不是很神奇？
    second = [[SecondVC alloc] init];
    [second.view setFrame:CGRectMake(0,CGRectGetMaxY(myScrollView.frame), width, height-CGRectGetHeight(myScrollView.frame))];
```
苹果已经给我写好切换UIViewController的transitionFromViewController方法了：
```OC
#pragma mark - 切换viewController
- (void)changeControllerFromOldController:(UIViewController *)oldController toNewController:(UIViewController *)newController
{
    [self addChildViewController:newController];
    /**
     *  切换ViewController
     */
    [self transitionFromViewController:oldController toViewController:newController duration:0.3 options:UIViewAnimationOptionCurveEaseIn animations:^{

        //做一些动画

    } completion:^(BOOL finished) {

        if (finished) {

            //移除oldController，但在removeFromParentViewController：方法前不会调用willMoveToParentViewController:nil 方法，所以需要显示调用
            [self didMoveToParentViewController:newController];
            [oldController willMoveToParentViewController:nil];
            [oldController removeFromParentViewController];
            currentVC = newController;

        }else
        {
            currentVC = oldController;
        }

    }];
}
```
写到这里大家对addChildViewController有一定的了解了，当一个界面比较复杂的时候我们就可以采用这种方式来降低耦合度（如果各位有更加好的方法，希望不要吝惜交流一下），这样做对页面的逻辑更加分明，如果有可以重用的也方便重用，而且View没有显示也不会load，减少内存的使用。
同时，还可以在一个parent ViewController上添加多个child ViewController，实际中这样的页面也是挺多的.
```OC
//在ViewController 中添加其他UIViewController
    FirstVC * first = [[FirstVC alloc] init];
    [self addChildViewController:first];
    //addChildViewController 会调用 [child willMoveToParentViewController:self] 方法，但是不会调用 didMoveToParentViewController:方法，官方建议显示调用
    [self didMoveToParentViewController:first];
    [first.view setFrame:CGRectMake(0, CGRectGetMaxY(myScrollView.frame), width, 300)];
    [self.view addSubview:first.view];

    SecondVC * second = [[SecondVC alloc] init];
    [self addChildViewController:second];
    [self didMoveToParentViewController:second];
    [second.view setFrame:CGRectMake(0,CGRectGetMaxY(first.view.frame), width, 300)];
    [self.view addSubview:second.view];
```



