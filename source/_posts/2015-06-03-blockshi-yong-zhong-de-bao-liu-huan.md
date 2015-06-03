---
layout: post
title: "Block使用中的保留环"
date: 2015-06-03 17:27:35 +0800
comments: true
categories: Block
keywords: Block,块

---
下面类提供了一套接口，调用者可由此从某个 URL 中下载数据。在启动获取器时，可设置 completion handler，这个块会在下载结束之后以回调方式执行。
	
		//  YWNetworkFetcher.h
		#import <Foundation/Foundation.h>

		typedef void (^YWNetworkFetcherCompletionHandler)(NSData *data);
		@interface YWNetworkFetcher : NSObject

		@property (nonatomic,strong,readonly) NSURL *url;

		- (id)initWithURL:(NSURL *)url;
		- (void)startWithCompletionHandler:(YWNetworkFetcherCompletionHandler)completion;
		@end
		
		//  YWNetworkFetcher.m
		#import "YWNetworkFetcher.h"

		@interface YWNetworkFetcher ()
		@property (nonatomic,strong,readwrite)NSURL *url;
		@property (nonatomic,copy)YWNetworkFetcherCompletionHandler completionHandle;
		@property (nonatomic,strong) NSData *downloadedData;
		@end

		@implementation YWNetworkFetcher

		- (id)initWithURL:(NSURL *)url{
		    if (self =[super init]) {
        		_url = url;
    		 }
    		return self;
		}
		- (void)startWithCompletionHandler:(YWNetworkFetcherCompletionHandler)completion{
    		self.completionHandle = completion;
    		// start the request
    		// request sets downloadedData property
    		// when request is finished,p_requestCompleted is called;
		}

		- (void)p_requestCompleted{
    		if (_completionHandle) {
        		_completionHandle(_downloadedData);
    		}
		}
		@end
	
	某个类会创建这个网络数据获取对象，并用其从 URL 中获取数据：
	
		@implementation YWClass {
    		YWNetworkFetcher *_networkFetcher;
    		NSData *_fetchedData;
		}


		- (void)downloadData{
    		NSURL *url = [[NSURL alloc] initWithString:@"http://wwww.xxx"];
    		_networkFetcher = [[YWNetworkFetcher alloc] initWithURL:url];
    		[_networkFetcher startWithCompletionHandler:^(NSData *data) {
        		NSLog(@"Request URL %@ finished",_networkFetcher.url);
        		_fetchedData = data;
    		}];
		}
		@end

	`这里面就存在一个保留环。`因为 completionHandle块 要设置 _fetchedData 实例变量，所以他必须捕获 self 变量。这就是说，handle 块保留了创建网络数据获取器的那个 YWClass 实例。而 YWClass 实例通过 strong 实例变量保留了获取器，最后，获取对象又保留了 handler 块。

	![保留环](http://7sbygq.com1.z0.glb.clouddn.com/liucheng.png)

	要打破保留环：要么令 _networkFetcher 实例变量不再引用获取器，要么令获取器的 completionHandle 属性不再持有 handler 块。在网络数据获取器这个例子中，应该等 completionHandle 执行完了，再去打破保留环，以便使获取器对象在 handler 块中爆出存活。比如， completionHandle 块的代码这样修改：
		
		[_networkFetcher startWithCompletionHandler:^(NSData *data) {
        	NSLog(@"Request URL %@ finished",_networkFetcher.url);
        	_fetchedData = data;
        	_networkFetcher = nil;
    	}];
    	
    如果让调用者自己来将获取器对箱保留存活的话，他们会觉得麻烦。大部分网络通讯库都采用下面这种方法：
    	
    	- (void)downloadData{
    		NSURL *url = [[NSURL alloc] initWithString:@"http://wwww.xxx"];
    		YWNetworkFetcher *networkFetcher = [[YWNetworkFetcher alloc] initWithURL:url];
    		[networkFetcher startWithCompletionHandler:^(NSData *data) {
        		NSLog(@"Request URL %@ finished",networkFetcher.url);
        		_fetchedData = data;
    		}];
		}
		
	然后就 YWNetworkFetcher 当前代码来看此做法会引用保留环。而此次比上面那个例子更难发现，completionHandle 通过获取器对象来引用其中的 URL。于是，块就要保留获取器对象，而获取器对象反过来又经由其 completionHandle 属性又保留了这个块。要修复这个问题也不难，回想一下获取器对象要把 completion handle 保存在属性里面，其唯一目的就是稍后使用这个块。然而 completionHandle 执行完后就没必要再保留它了。所以，只需把 p_requestCompleted 代码修改一下就好：
	
		- (void)p_requestCompleted{
    		if (_completionHandle) {
        		_completionHandle(_downloadedData);
    		}
    		self.completionHandle = nil;
		}
	这样保留环就解除了。`一定要找个适当的机会解除保留环，而不是把责任推给 API 调用者。`
