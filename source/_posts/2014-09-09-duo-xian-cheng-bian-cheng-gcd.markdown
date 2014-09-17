---
layout: post
title: "多线程编程-GCD"
date: 2014-09-09 21:41:09 +0800
comments: true
categories: 
---
*注释：本文内容大多来自《Objective-C高级编程 iOS和OS X多线程和内存管理》这本书*

### Dispatch Queue

官方说明：**开发者要做的只是定义向执行的任务并追加到适当的 Dispatch Queue**,这句话的源码表示如下：

	dispatch_async(queue, ^{
        /**
         *  想执行的任务
         */
    });
   
Dispatch Queue 是执行处理的等待队列。在 Block 语法中记述想执行的处理并追加在 Dispatch Queue 中，Dispatch Queue 按照 FIFO 顺序执行处理。

另外，在执行处理时有两种 Dispatch Queue，一种等待现在执行中处理的 Serial Dispatch Queue，另一种是不等待现在执行中处理的 Concurrent Dispatch Queue。

#### dispatch_queue_create

* 通过 GCD 的 API 生成 Dispatch Queue

	dispatch_queue_t mySerialQueue  = dispatch_queue_create("com.demo.gcd.gcdDemo", NULL);
    dispatch_queue_t myConcurrentQueue = dispatch_queue_create("com.demo.gcd.gcdDemo", DISPATCH_QUEUE_CONCURRENT);
    
dispatch_queue_create 第一个参数指定 Queue 的名称，该名称在 Xcode 和 Instruments 的调试器中作为 Dispatch Queue 的名称表示，可以设置为 Null，但是为了易于调试，最好按照苹果推荐的用应用 ID 的逆序全程域名表示。

这里需注意ios6以下 GCD 对象没有被纳入 ARC 的管理范围，所以对于 dispatch_queue_create 和 dispatch_retain 生成的 Dispatch Queue 需要手动释放。
	
	# if __IPHONE_OS_VERSION_MIN_REQUIRED< 60000
    dispatch_retain(mySerialQueue);
    dispatch_release(mySerialQueue);
    dispatch_release(mySerialQueue);
	# endif
	
#### Main Dispatch Queue / Global Dispatch Queue
* 通过或地区系统标准提供的 Dispatch Queue。

系统提供两种 Main Dispatch Queue / Global Dispatch Queue,分别是 Serial Dispatch Queue
 和 	Concurrent Dispatch Queue。Global Dispatch Queue 有 4 个执行优先级，分别是高优先级（High Priority）、默认优先级（Default Priority）、低优先级（Low Priority）和后台优先级（Background Priority）。
 
 
	dispatch_queue_t mainDispatchQueue = dispatch_get_main_queue();
    dispatch_queue_t globalDispatchQueue =
    dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
另外，对于Main Dispatch Queue 和 Global Dispatch Queue 执行 dispatch_main 和 dispatch_release 函数不会引起任何变化，也不会有任何问题。

#### dispatch_set_target_queue

dispatch_create 函数生成的 Dispatch Queue 使用与默认优先级 Globle Dispatch Queue 相同的优先级。在后台执行处理的 Serial Dispatch Queue 的生成方法：

	dispatch_queue_t mySerialQueue  = dispatch_queue_create("com.demo.gcd.gcdDemo", NULL);
	dispatch_queue_t globalDispatchQueueBackground = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0);
    dispatch_set_target_queue(mySerialQueue, globalDispatchQueueBackground);
    
指定 mySerialQueue 和 globalDispatchQueueBackground 相同优先级。如果第一个参数指定为系统提供的 Main Dispatch Queue 和 Global Dispatch Queue 则不知道会出现什么状况，因为这些均不可指定。

#### dispatch_after 
想在指定时间后执行处理的情况，可使用 dispatch_after 函数来实现。

	dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3ull * NSEC_PER_SEC);
    dispatch_after(time, dispatch_get_main_queue(), ^{
        NSLog(@"waited at least three seconds");
    });
  
因为 Main Dispatch Queue 在主线程的 RunLoop 中执行，所以比如每隔 1/60 秒执行的 RunLoop 中，blick 最快在 3 秒后执行，最慢在3秒+1/60秒后执行，并且在Main Dispatch Queue 有大量处理追加或主线程的处理本身有延迟时，这个时间更长。

dispatch_time_t 类型的值为第一个参数，DISPATCH_TIME_NOW 表示现在，ull 表示（unsigned long long）,NSEC_PER_SEC 表示以秒为单位计算。如果想用 dispatch_after 指定 2014年11月11日11时11分11秒这个时间执行 Block,则要用到 dispatch_walltime 函数使用 struct timespec 类型的时间得到 dispatch_time_t 类型的值。 struct timespec 类型的时间可以很轻松的通过 NSDate 类对象生成。

	dispatch_time_t getDispatchTimeByDate (NSDate *date){
    NSTimeInterval interval;
    double second, subsecond;
    struct timespec time;
    dispatch_time_t milestone;
    
    interval = [date timeIntervalSince1970];
    subsecond = modf(interval, &second);
    time.tv_sec = second;
    time.tv_nsec = subsecond * NSEC_PER_SEC;
    milestone = dispatch_walltime(&time, 0);
    
    return milestone;
	}
	
#### Dispatch Group

在追加到 Concurrent Dispatch Queue 中的多个处理全部结束后想执行结束处理，这时需要用到 dispatch group。下面追加 3 个 Block 到 Global Dispatch Queue，这些 Block 如果全部执行完毕，就会执行 Main Dispatch Queue 中结束处理用的 Block。

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"blk0");
    });
    dispatch_group_async(group, queue, ^{
        NSLog(@"blk1");
    });
    dispatch_group_async(group, queue, ^{
        NSLog(@"blk2");
    });
    dispatch_group_notify(group, dispatch_get_main_queue() , ^{
        NSLog(@"done");
    });

	blk0
	blk2
    blk1
 	done
 
dispatch_group_async 函数和 dispatch_async 函数相同，都追加 Block 到指定的 Dispatch Queue 中。指定的 Block 属于指定的 Dispatch Group。

#### dispatch_barrier_async
为了高效的访问数据库，读取处理追加到 Concurrent Dispatch Queue 中，写入处理在任一个读取处理没有执行的状态下，追加到 Serial Dispatch Queue 中即可（在写入处理结束之前，读取处理不可执行）。
GCD 为我们提供了 dispatch_barrier_async 函数。该函数同 dispatch_queue_create 函数生成的 Concurrent Dispatch Queue 一起使用。

#### dispatch_sync
dispatch_sync 函数将指定的 Block "同步"追加到指定的 Dispatch Queue 中。在追加 Blcok 结束之前，dispatch_sync 函数会一直等待。dispatch_sync 使用简单，但是容易引起死锁。下面两个源代码都会死锁。

	dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_sync(mainQueue, ^{ NSLog(@"Hello?"); });
    
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_async(mainQueue, ^{
        dispatch_sync(mainQueue, ^{
            NSLog(@"Hello?");
        });
    });

#### dispatch_apply
dispatch_apply 函数是 dispatch_sync 函数和 Dispatch Group 的关联 API。该函数按指定的次数将指定的 Block 追加到指定的 Dispatch Queue 中，并等待全部执行结束。

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	dispatch_apply(10, queue, ^(size_t index) {
        NSLog(@"%zu",index);
    });
    NSLog(@"done");

因为 dispatch_apply 函数也与 dispatch_sync 函数相同，会等待处理执行结束，因此推荐在 dispatch_async 函数中非同步地执行 dispatch_apply 函数。

	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	dispatch_async(queue, ^{
        dispatch_apply(10, queue, ^(size_t index) {
            NSLog(@"%zu",index);
        });
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"done");
        });
    });
 
#### dispatch_suspend / dispatch_resume
当追加大量处理到 Dispatch Queue 时，在追加处理的过程中，有时希望不执行已追加的处理。例如演算结果被 Block 截获时，一些处理会对这个演算结果造成影响。这种情况，只要挂起 Dispatch Queue 即可。当可以执行时再恢复。 dispatch_suspend 挂起，dispatch_resume 恢复。

#### dispatch I/O
在读取较大文件时，如果将文件分成合适的大小并使用 Global Dispatch Queue 并列读取的话，应该会比一般的读取速度快不少。能实现这一共跟你过的就是 Dispatch I/O 和 Disaptch Data。

		