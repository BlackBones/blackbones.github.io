---
layout:     post
title:      WKWebView与JS
subtitle:   GCD 
date:       2017-04-28
author:     BlackBones
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - WKWebView  JavaScript
---
今天来总结一下WKWebView与JS的交互。
# 一、拦截跳转的URl
这种方法实际上并未涉及OC与JS的交互，只是在WebView在发生跳转之前，拦截将要跳转的URL，根据判断URL，来实现响应的操作。

```
//WKNavigationDelegate
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    NSString *url = navigationAction.request.URL.absoluteString;
    if ([url rangeOfString:@"相应的URL"].location != NSNotFound) {
        //跳转原生界面
        //Cancel the navigation
        decisionHandler(WKNavigationActionPolicyCancel);
        return;
    }
    decisionHandler(WKNavigationActionPolicyAllow);
}
```

# 二、OC调用JS
很简单，通常调用JS的方法，在网页加载成功的时候，调用一些JS代码对网页进行设置。

```
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    //设置JS
    NSString *JSString = @"document.getElementsByName('input')[0].attributes['value'].value";
    //执行JS
    [webView evaluateJavaScript:JSString completionHandler:^(id _Nullable response, NSError * _Nullable error) {
        NSLog(@"value: %@ error: %@", response, error);
    }];
}
```
在运行时也可以加载JS代码

```
NSString *js = @"I am JS Code";

//初始化WKUserScript对象
//WKUserScriptInjectionTimeAtDocumentEnd为网页加载完成时注入
WKUserScript *script = [[WKUserScript alloc] initWithSource:js injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];

//根据生成的WKUserScript对象，初始化WKWebViewConfiguration
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
[config.userContentController addUserScript:script];

//初始化WebView
self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config];
[_wkWebView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"URL"]]];
[self.view addSubview:_webView];
```
上边假如有一些JS操作执行,就触发WKWebView的另一个代理WKUIDelegate的回调方法.

# 三、JS调用OC
当JS端想传一些数据给iOS.那JS端会调用下方方法来发送.

```
window.webkit.messageHandlers.<OC方法名>.postMessage(<数据>)
```
那么在OC中的处理方法如下.它是WKScriptMessageHandler的代理方法.name和上方JS端中的方法名相对应.

```
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
```
实际代码：

```
//设置addScriptMessageHandler与name.并且设置<WKScriptMessageHandler>协议与协议方法
[[_webView configuration].userContentController addScriptMessageHandler:self name:@"方法名"];

//WKScriptMessageHandler协议方法
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    //根据message.body可以获取JS端发送的数据
}

```














