---
layout: post
title: "Block 带有自动变量的匿名函数"
date: 2014-09-06 10:15:28 +0800
comments: true
categories: 
---

*注释：本文内容大多来自《Objective-C高级编程 iOS和OS X多线程和内存管理》这本书*

###什么是Block
Block 是带有自动变量（局部变量）的匿名函数。匿名函数就是“没有名称的函数”，这里不做详细解释。这里主要讨论“带有自动变量”究竟是什么？首先了解下函数中使用的变量，如下：

变量名			 | 函数调用可传递值
--------------	 | ---------------
自动变量（局部变量）| NO
函数的参数		 | NO
静态变量			 | YES
静态全局变量		 | YES
全局变量 			 | YES	

静态变量、静态全局变量和全局变量，虽然这些变量的作用域不同，但在整个程序中，一个变量总保持在一个内存区域。因此，虽然多次调用函数，但该变量值总保持不变。	

	int buttonId = 0;

	//使用全局变量
	-(void)buttonCallBack:(int)event{
    	NSLog(@"buttonId:%d event=%d\n",buttonId,event);
	}
一个按钮运行正常，如果存在多个按钮呢？

	//使用全局变量
	-(void)buttonCallBack:(int)event{
   		NSLog(@"buttonId:%d event=%d\n",buttonId,event);
	}

	for (int i = 0 ; i<BUTTON_MAX; ++i) {
        buttonId = i;
        [self buttonCallBack:BUTTON_IDOFFSET+i];
    }
    
 该代码的问题非常明显。全局变量buttonId只有一个，所有回调都使用for循环最后的值。当然如果不使用全局变量，回调函数把buttonId作为函数参数传递，就能解决问题。
 
 	//使用函数参数
	-(void)buttonCallBack:(int)buttonId event:(int)event{
    	NSLog(@"buttonId:%d event=%d\n",buttonId,event);
	}
	
但是，回调函数在保持event以外，还要保持回调方的按钮Id。Object-C 使用类也可保持变量值且能多次持有该变量自身，实现如下：
	
	@interface ButtonCallbackObject : NSObject
	{
    	int _buttonId;
	}
	@end
	@implementation ButtonCallbackObject
	- (id)initWithButtonId:(int)buttonId{
    	self = [super init];
    	_buttonId = buttonId;
    	return self;
	}

	-(void)callback:(int)event{
    	NSLog(@"buttonId:%d event=%d\n",_buttonId,event);
	}

	@end
	
如果使用该类，只需保持对象即可保持按钮的id，但声明和实现类无疑增加了代码的长度。
这是我们就要用到Block了。Block提供了类似Object-C类生成实例或对象来保持变量值的方法。（如果不理解，可仔细回味这句话）。 下面使用Block来实现：

	  for (int i = 0 ; i<BUTTON_MAX; ++i) {

        [self setButtonCallBackBlock:^(int event) {
            NSLog(@"buttonId:%d event=%d\n",i,event);
        }];
        _ButtonCallBackBlock(i+BUTTON_IDOFFSET);
   	  }
   	  
该源代码将“带有自动变量i值的匿名函数”设定为函数的回调。像这样，不使用静态变量，全局变量，不声明Object-C 类，使用Block 即可使用带有自动变量值的匿名函数。

*这一篇讲的概念就是block可截获自动变量值*
