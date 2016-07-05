---
title: WebViewJavascriptBridge分析
date: 2016-05-06 09:14:12
tags: Object-C
---
 UIWebview/WebViews 在 Obj-C 和 JavaScript 之间通信的桥。虽然网上有很多介绍如何使用的文章，但是推荐大家到 [github-WebViewJavaScriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge) 上看最新版本的介绍和使用。这里我们来看看它的实现过程。
 
## 通信基础
首先我们需要知道 UIWebView 和 JS 通信的两个基础方法：1. webView 的协议方法：

``` Obj-C
- (BOOL)webView:(UIWebView *)webView
shouldStartLoadWithRequest:(NSURLRequest *)request
 navigationType:(UIWebViewNavigationType)navigationType; 
```
在 web view 加载一个 frame 之前调用，可以开始加载返回 Yes，反之 No。所以这个方法可用于接收 js 里面发送的请求。我们可以根据请求的方法与参数来做业务处理。

2：

``` Obj-C
- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;
```
oc 调用 js 的方法返回执行后的值。

## 通信过程
知道了 oc 和 js 的通信基础，我们来看看 WebViewJavaScriptBridge 的通信过程。建造一座桥首先要有桥头、桥尾。“桥头”是 webView，“桥尾”是 JSCode.

``` Obj-C
+ (instancetype)bridgeForWebView:(WVJB_WEBVIEW_TYPE*)webView;
+ (instancetype)bridgeForWebView:(WVJB_WEBVIEW_TYPE*)webView {
    WebViewJavascriptBridge* bridge = [[self alloc] init];
    [bridge _platformSpecificSetup:webView];
    return bridge;
}
- (void) _platformSpecificSetup:(WVJB_WEBVIEW_TYPE*)webView {
    _webView = webView;
    _webView.delegate = self;
    _base = [[WebViewJavascriptBridgeBase alloc] init];
    _base.delegate = self;
}
```

bridge 对象生成时需传入 webView，这里生成“桥头”设置其 delegate 为 self，并创建 base 对象同样设置 delegate 为 self。关于 base 后面再说，我们接着向下看。base 在初始化时创建了两个可变字典 messageHandlers 、responseCallbacks ，一个可变数组 startupMessageQueue，一个_uniqueId。

“桥头”构建好了，我们要给它注册一些操作者来处理从“桥尾”发起的请求并把处理后的数据返回给“桥头”，同时给“桥头”一个回调的方法。

``` Obj-C
- (void)registerHandler:(NSString*)handlerName handler:(WVJBHandler)handler;
```
调用此方法把处理者 handlerName 放到 messageHandlers，处理后的结果放到 handler 里面。

``` Obj-C
_base.messageHandlers[handlerName] = [handler copy];
```

“桥头”不能只接收而不发消息，也需要给它主动发消息的能力：

``` Obj-C
- (void)callHandler:(NSString*)handlerName;
- (void)callHandler:(NSString*)handlerName data:(id)data;
- (void)callHandler:(NSString*)handlerName data:(id)data responseCallback:(WVJBResponseCallback)responseCallback;
```

调用三个方法中的任意一个都是执行 base 的 sendData 方法，在 sendData 方法里把传入的数据封装成一个 message（dictionary）用于转化为 json 字符串传给 js，js 再执行解析和对应的处理。这里有一个地方需要注意：如果在webView 还没有加载 url 之前我们调用了 callHandler 怎么办呢？作者在 base 对象里生成了 startupMessageQueue，并在我们执行 sendData 时判断 startupMessageQueue 是否存在，存在就把 message 添加到 startupMessageQueue 中，在 webView 注册 javascriptFile 时执行“桥头”发过来的消息。再把 startupMessageQueue 设为 nil。 startupMessageQueue 就完成了它的使命，再次复活就只能等待 reset 重启。

``` Obj-C
- (void)_queueMessage:(WVJBMessage*)message {
    if (self.startupMessageQueue) {
        [self.startupMessageQueue addObject:message];
    } else {
        [self _dispatchMessage:message];
    }
}
- (void)injectJavascriptFile {
    NSString *js = WebViewJavascriptBridge_js();
    [self _evaluateJavascript:js];
    if (self.startupMessageQueue) {
        NSArray* queue = self.startupMessageQueue;
        self.startupMessageQueue = nil;
        for (id queuedMessage in queue) {
            [self _dispatchMessage:queuedMessage];
        }
    }
}
```

WebViewJavascriptBridge_js 这个扩展类可看做是“桥尾”的封装，主要是实现一些方法，用于使“桥头”和“桥尾”之间的通信在代码实现上更加的简单、方便。

我们在使用中发现“桥尾”给“桥头”发消息并没有刷新页面，oc 的webview 的协议方法是怎么接收到的呢？

``` JavaScript
messagingIframe = document.createElement('iframe');
messagingIframe.style.display = 'none';
messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://' + QUEUE_HAS_MESSAGE;
document.documentElement.appendChild(messagingIframe);
```
在 js 中创建一个隐藏的 iframe，每次发消息更新其 src 即可使 webView 的协议方法执行。