---
layout: post
title: "iOS小Demo--图片无限轮播"
date: 2015-12-05 
categories: iOS
comments: false
tags: OC 
---

网上已经有很多无限循环的小Demo,这里我把感觉还不错的做了整理,不多说直接上代码.
<!-- more -->
```
#import "RootViewController.h"
#define screenWidth [[UIScreen mainScreen] bounds].size.width // 屏幕宽度的宏定义
@interface RootViewController ()<UIScrollViewDelegate>
@property (nonatomic,retain)UIScrollView *scrollView ;
@property (nonatomic,retain)UIPageControl *pageControl ;
@property (nonatomic,retain)NSMutableArray *pictureNameArray ; // 存放图片名称
@property (nonatomic,assign)int arrayIndex ; //存放数组下标  默认是0
@property (nonatomic,retain)NSTimer *timer ; // 定时器,根据定时器时间触发事件,达到图片无限滚动的效果
@end
```
**思路 :我们始终只显示中间那个imageView,每次偏移量改变之后我们又会重新把它设为中间位置的偏移量.
我们所要做的只是改变三个imageView中存放的图片.
表面看我们是滑动了scrollView,其实偏移量并没有改变(实际是先改变,改变了之后有设置回来),始终停留在中间那个imageView上.**
```
@implementation RootViewController
- (void)viewDidLoad
{
    [super viewDidLoad];
    self.navigationItem.title = @"无限滚动图片" ;
    // 往数组里添加图片名称
    self.pictureNameArray = [NSMutableArray arrayWithArray:@[@"1.jpg",@"2.jpg",@"3.jpg"]] ;
    // 防止UIScrollView位置发生偏移
    self.automaticallyAdjustsScrollViewInsets = NO ;
    [self.view addSubview:self.scrollView] ;
    [self.view addSubview:self.pageControl] ;
    [self creatImageViewForScrollView] ;
    [self creatTimer] ;
}
// scrollView的懒加载
- (UIScrollView *)scrollView
{
    if (!_scrollView)
    {
        _scrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 64, SCREENWIDTH, 200)] ;
        // 设置可滑动区域
        _scrollView.contentSize = CGSizeMake(SCREENWIDTH*self.pictureNameArray.count, 0) ;
        // 设置初始偏移量
        _scrollView.contentOffset = CGPointMake(SCREENWIDTH, 0) ;
        // 设置能否分页
        _scrollView.pagingEnabled = YES ;
        // 隐藏水平滚动条
        _scrollView.showsHorizontalScrollIndicator = NO ;
        _scrollView.delegate = self ;
    }
    return _scrollView ;
}
// pageControl的懒加载
- (UIPageControl *)pageControl
{
    if (!_pageControl)
    {
        CGFloat floatX = self.view.bounds.size.width - 150 ; // 距离屏幕右侧150处
        CGFloat floatY = 64 + 200 - 50 ; // 距离scrollView最底部50处
        _pageControl = [[UIPageControl alloc] initWithFrame:CGRectMake(floatX, floatY, 150, 50)] ;
        // 设置小圆点的个数
        _pageControl.numberOfPages = self.pictureNameArray.count ;
        // 设置当前页
        _pageControl.currentPage = 0 ;
        // 设置未被选中时小圆点颜色
        _pageControl.pageIndicatorTintColor = [UIColor redColor] ;
        // 设置被选中时小圆点颜色
        _pageControl.currentPageIndicatorTintColor = [UIColor whiteColor] ;
        // 默认不能手动点小圆点改变页数
        _pageControl.enabled = NO ;
        // 把导航条设置为半透明状态
        [_pageControl setBackgroundColor:[[UIColor blackColor] colorWithAlphaComponent:0.5]] ;
    }
    return _pageControl ;
}
```
```
// 定义一个往scrollView上添加imageView的方法
- (void)creatImageViewForScrollView
{  
    for (int i = 0; i < self.pictureNameArray.count; i++)
    {
        UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(SCREENWIDTH * i, 0, SCREENWIDTH, 200)] ;
        // 添加tag值
        imageView.tag = 1000 + i ;
        // 给初始的imageView设置默认图片
        // 判断i值,i对应存放图片名称的数组下标
        // 因为只有三张图片,我们始终只显示中间的imageView,初始时scrollView的偏移量已经在中间,因此我们把第二张,也就是数组中下标为1的图片名称放在中间的imageView上. 2--0--1
        if (i == 0)
        {
            imageView.image = [UIImage imageNamed:[self.pictureNameArray lastObject]] ;
        }
        else if (i == 1)
        {
            imageView.image = [UIImage imageNamed:[self.pictureNameArray firstObject]] ;
        }
        else if (i == 2)
        {
            imageView.image = [UIImage imageNamed:self.pictureNameArray[1]] ;
        }
        [self.scrollView addSubview:imageView] ;
    }
}

// 生成定时器,当定时器所要执行的方法触发时,改变scrollView偏移量
- (void)creatTimer
{
    self.timer = [NSTimer scheduledTimerWithTimeInterval:3 target:self selector:@selector(changeScrollViewContentOffset) userInfo:nil repeats:YES] ;
}

- (void)changeScrollViewContentOffset
{
    // 每次给scrollView加一个屏幕的偏移量 向右无限滚动
    [self.scrollView setContentOffset:CGPointMake(self.scrollView.contentOffset.x + SCREENWIDTH, 0) animated:YES] ;
}
```
```
// 当设置偏移量并且带动画效果的时候才会执行该方法
- (void)scrollViewDidEndScrollingAnimation:(UIScrollView *)scrollView
{
    // 如果是无限向右滚动,由于我们初始时偏移量就在中间,因此一直会执行该段代码
    if (self.scrollView.contentOffset.x > SCREENWIDTH)
    {
        // 如果当前下标是数组最后元素的下标,说明图片已经滚动到最后一张,这是需要重新从第一张开始 否则下标加一
        if (self.arrayIndex == self.pictureNameArray.count - 1)
        {
            self.arrayIndex = 0 ;
        }
        else
        {
            self.arrayIndex ++ ;
        }
    }
    // 如果是无限向左滚动
    else
    {
        if (self.arrayIndex == 0)
        {
            self.arrayIndex = (int)self.pictureNameArray.count - 1 ;
        }
        else
        {
            self.arrayIndex -- ;
        }
    }
    // 当我们计算好数组下标之后,把scrollView的偏移量再重新设置回中间
    [self.scrollView setContentOffset:CGPointMake(SCREENWIDTH, 0) animated:NO] ; // 此时不用带动画效果
    // 设置pageControl的当前页
    self.pageControl.currentPage = self.arrayIndex ;
    // 改变imageView中三张图片的位置.把中间的imageView始终显示下标为arrayIndex的图片
    [self changeImage:self.arrayIndex] ;
}

// 当我们手动拖拽时 先将定时器干掉,否则计时器时间到又会滚动图片.
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    [self.timer invalidate] ;
    self.timer = nil ;
}
// 当手动拖拽结束时 再开启一个新的计时器
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    [self creatTimer] ;
}

// 当减速结束时,改变偏移量,并改变需要显示的图片
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    // 如果是无限向右滚动,由于我们初始时偏移量就在中间,因此一直会执行该段代码
    if (self.scrollView.contentOffset.x > SCREENWIDTH)
    {
        // 如果当前下标是数组最后元素的下标,说明图片已经滚动到最后一张,这是需要重新从第一张开始 否则下标加一
        if (self.arrayIndex == self.pictureNameArray.count - 1)
        {
            self.arrayIndex = 0 ;
        }
        else
        {
            self.arrayIndex ++ ;
        }
    }
    // 如果是无限向左滚动
    else
    {
        if (self.arrayIndex == 0)
        {
            self.arrayIndex = (int)self.pictureNameArray.count - 1 ;
        }
        else
        {
            self.arrayIndex -- ;
        }
    }
    // 当我们计算好数组下标之后,把scrollView的偏移量再重新设置回中间
    [self.scrollView setContentOffset:CGPointMake(SCREENWIDTH, 0) animated:NO] ; // 此时不用带动画效果
    // 设置pageControl的当前页
    self.pageControl.currentPage = self.arrayIndex ;
    // 改变imageView中三张图片的位置.把中间的imageView始终显示下标为arrayIndex的图片
    [self changeImage:self.arrayIndex] ;
}
```
```
// 改变imageView显示的图片名称
- (void)changeImage : (int)index
{
    // 首先取到三个imageView
    //提取imageView
    UIImageView *imageView1 = (UIImageView *)[self.scrollView viewWithTag:1000];
    UIImageView *imageView2 = (UIImageView *)[self.scrollView viewWithTag:1001];
    UIImageView *imageView3 = (UIImageView *)[self.scrollView viewWithTag:1002];


    if (index == self.pictureNameArray.count - 1)
    {
        imageView2.image = [UIImage imageNamed:self.pictureNameArray[index]];
        imageView3.image = [UIImage imageNamed:self.pictureNameArray[0]];
        imageView1.image = [UIImage imageNamed:self.pictureNameArray[index-1]];
    }
    else if (index == 0)
    {
        imageView2.image = [UIImage imageNamed:self.pictureNameArray[index]];
        imageView3.image = [UIImage imageNamed:self.pictureNameArray[1+index]];
        imageView1.image = [UIImage imageNamed:self.pictureNameArray.lastObject];
    }
    else
    {
        imageView2.image = [UIImage imageNamed:self.pictureNameArray[index]];
        imageView3.image = [UIImage imageNamed:self.pictureNameArray[index+1]];
        imageView1.image = [UIImage imageNamed:self.pictureNameArray [index-1]];
    }

}
@end
```




