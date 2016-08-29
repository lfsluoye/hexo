---
layout: post
title: "使用WKWebView替换UIWebView"
date: 2015-11-18 
categories: iOS
comments: false
tags: Swift
---
> 开发App的过程中，常常会遇到在App内部加载网页，通常用UIWebView加载。这个自iOS2开始使用的网页加载器一直是开发的心病：加载速度慢，占用内存多，优化困难。如果加载网页多，还可能因为过量占用内存而给系统kill掉。各种优化的方法效果也不那么明显。

iOS8以后，苹果推出了新框架Wekkit，提供了替换UIWebView的组件WKWebView。各种UIWebView的问题没有了，速度更快了，占用内存少了，一句话，WKWebView是App内部加载网页的最佳选择！

先看下 WKWebView的特性：
>* 在性能、稳定性、功能方面有很大提升（最直观的体现就是加载网页是占用的内存，模拟器加载百度与开源中国网站时，WKWebView占用23M，而UIWebView占用85M）
* 允许JavaScript的Nitro库加载并使用（UIWebView中限制）
* 支持了更多的HTML5特性
* 高达60fps的滚动刷新率以及内置手势
* 将UIWebViewDelegate与UIWebView重构成了14类与3个协议
<!-- more -->

然后从以下几个方面说下WKWebView的基本用法
>* 加载网页
* 加载的状态回调
* 新的WKUIDelegate协议
* 动态加载并运行JS代码
* webView 执行JS代码
* JS调用App注册过的方法

### 一、加载网页
加载网页或HTML代码的方式与UIWebView相同，代码示例如下：
```
 WKWebView *webView = [[WKWebView alloc] initWithFrame:self.view.bounds];
[webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
[self.view addSubview:webView];
```
### 二、加载的状态回调 （WKNavigationDelegate）
用来追踪加载过程（页面开始加载、加载完成、加载失败）的方法：
```
// 页面开始加载时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation;
// 当内容开始返回时调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation;
// 页面加载完成之后调用
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation;
// 页面加载失败时调用
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation;
```
页面跳转的代理方法：
```
// 接收到服务器跳转请求之后调用
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation;
// 在收到响应后，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler;
// 在发送请求之前，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;
```
### 三、新的WKUIDelegate协议
这个协议主要用于WKWebView处理web界面的三种提示框(警告框、确认框、输入框)，下面是警告框的例子:
```
/**
 *  web界面中有弹出警告框时调用
 *
 *  @param webView           实现该代理的webview
 *  @param message           警告框中的内容
 *  @param frame             主窗口
 *  @param completionHandler 警告框消失调用
 */
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(void (^)())completionHandler;
```
### 四、动态加载并运行JS代码
用于在客户端内部加入JS代码，并执行，示例如下：
```
// 图片缩放的js代码
NSString *js = @"var count = document.images.length;for (var i = 0; i < count; i++) {var image = document.images[i];image.style.width=320;};window.alert('找到' + count + '张图');";
// 根据JS字符串初始化WKUserScript对象
WKUserScript *script = [[WKUserScript alloc] initWithSource:js injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];
// 根据生成的WKUserScript对象，初始化WKWebViewConfiguration
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
[config.userContentController addUserScript:script];
_webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config];
[_webView loadHTMLString:@"<head></head><imgea src='http://www.nsu.edu.cn/v/2014v3/img/background/3.jpg' />"baseURL:nil];
[self.view addSubview:_webView];
```
### 五、webView 执行JS代码
用户调用用JS写过的代码，一般指服务端开发的：
```
//javaScriptString是JS方法名，completionHandler是异步回调block
[self.webView evaluateJavaScript:javaScriptString completionHandler:completionHandler];
```
### 六、JS调用App注册过的方法
再WKWebView里面注册供JS调用的方法，是通过WKUserContentController类下面的方法：
```
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
```
scriptMessageHandler是代理回调，JS调用name方法后，OC会调用scriptMessageHandler指定的对象。

JS在调用OC注册方法的时候要用下面的方式：
```
window.webkit.messageHandlers.<name>.postMessage(<messageBody>)
```
注意，name(方法名)是放在中间的，messageBody只能是一个对象，如果要传多个值，需要封装成数组，或者字典。整个示例如下：
```
//OC注册供JS调用的方法
[[_webView configuration].userContentController addScriptMessageHandler:self name:@"closeMe"];

//OC在JS调用方法做的处理
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    NSLog(@"JS 调用了 %@ 方法，传回参数 %@",message.name,message.body);
}

//JS调用
    window.webkit.messageHandlers.closeMe.postMessage(null);
```
如果你在self的dealloc打个断点，会发现self没有释放！这显然是不行的！谷歌后看到一种解决方法，如下：
```
@interface WeakScriptMessageDelegate : NSObject<WKScriptMessageHandler>

@property (nonatomic, weak) id<WKScriptMessageHandler> scriptDelegate;

- (instancetype)initWithDelegate:(id<WKScriptMessageHandler>)scriptDelegate;

@end

@implementation WeakScriptMessageDelegate

- (instancetype)initWithDelegate:(id<WKScriptMessageHandler>)scriptDelegate
{
    self = [super init];
    if (self) {
        _scriptDelegate = scriptDelegate;
    }
    return self;
}

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    [self.scriptDelegate userContentController:userContentController didReceiveScriptMessage:message];
}

@end
```
思路是另外创建一个代理对象，然后通过代理对象回调指定的self
```
WKUserContentController *userContentController = [[WKUserContentController alloc] init];    
[userContentController addScriptMessageHandler:[[WeakScriptMessageDelegate alloc] initWithDelegate:self] name:@"closeMe"];
```
运行代码，self释放了，WeakScriptMessageDelegate却没有释放啊啊啊！
还需在self的dealloc里面 添加这样一句代码：
```
[[_webView configuration].userContentController removeScriptMessageHandlerForName:@"closeMe"];
```


