---
layout: post
title: "iOS-textView文本换行高度自动适应"
date: 2015-1-25 
comments: false
tags: ios 
---
由于开发中需要需要个类似QQ空间的一个评论功能,首先要做的输入框肯定是会换行的,因为iOS中(TextField)是单行组件,所以只能使用(TextView)这个多行组件了,可是在不做处理的情况下这个组件的文字在换行的时候会顶出输入框

解决思路:
<!-- more -->
####1.通过Block 在TextView代理方法中把文字内容的ContentSize回调
    改变根据文字改变TextView的高度
```OC
.h
typedef void (^ContentSizeBlock)(CGSize contentSize);

@interface ReplyInputView : UIView

@property (nonatomic,weak)UITextView *textfd;

-(void)setContentSizeBlock:(ContentSizeBlock) block;

@end
```
```
.m
#pragma mark - textView delegate

-(void)textViewDidChange:(UITextView *)textView

{//根据字长度判断是否隐藏占位符Label

self.lblPlaceholder.hidden = (textView.text.length > 0);

CGSize contentSize = self.textfd.contentSize;

self.sizeBlock(contentSize);

}
```
####2.在Vc中实例化我们输入框View的时候使用block调用更新约束的方法
```OC
#pragma mark - 初始化textview

-(void)initReplyInputView{

ReplyInputView *replyInputView =

[[ReplyInputView alloc]initWithFrame:

CGRectMake(0,KDeviceH-42-64, KDeviceW, 42) andAboveView:self.view];

replyInputView.textdelegate = self;

[self.view  addSubview:replyInputView];

self.replyInputView = replyInputView;

__weak typeof(self) weakSelf = self;

//使用block

[replyInputView setContentSizeBlock:^(CGSize contentSize) {

//更新约束

[weakSelf updateHeight:contentSize];

}];

}
```
    更新replyView的高度约束
```OC
-(void)updateHeight:(CGSize)contentSize

{

float height = contentSize.height +20;

CGRect frame = self.replyInputView.frame;

frame.origin.y -= height - frame.size.height;  //高度往上拉伸

frame.size.height = height;

self.replyInputView.frame = frame;

}
```
这样即可实现了



