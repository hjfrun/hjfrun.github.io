---
layout:     post
title:      iOS多线程（4）【GCD篇】
subtitle:   semaphore，线程同步
date:       2017-04-01 00:40:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - 线程同步
    - GCD
    - semaphore
---

### 简介

多线程编程中很重要的一个操作是线程同步。使用GCD进行多线程开发经常会碰到线程同步的情况。

dispatch_semaphore信号量就是为了解决多线程并发控制而产生的。`信号量`是一个整形值并且具有初始计数值，支持两个操作，加1和减1。分别对应signal操作和wait操作。大学期间学过操作系统课程的应该对这两个操作非常熟悉，这两个操作也叫做PV操作。当一个信号量被通知，即signal，其计数值会增加。当一个线程在一个信号量上等待，线程会被阻塞（如果有必要的话），直至计数器大于0，然后线程会减少这个数，继续执行。

在GCD中有三个semaphore操作相关的操作，分别是：`dispatch_semaphore_create`创建一个信号量、`dispatch_semaphore_signal`信号量加1、`dispatch_semaphore_wait`等待信号。第一个函数有一个整形的参数，我们可以理解为信号的总量，`dispatch_semaphore_signal`是发送一个信号，让信号量总数加1，`dispatch_semaphore_wait`等待信号，当信号总量少于0的时候就会一直等待，否则就可以正常执行，并让信号量总量减1，根据这样的原理，我们便可以快速的创建一个并发控制来同步任务和有限资源访问控制。`semaphore`比`栅栏函数`更细粒度的控制。

##### 代码示例1：

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 创建信号量
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);

NSMutableArray *arrayM = [NSMutableArray array];

dispatch_apply(10000, queue, ^(size_t i) {
    // 数据进入，等待处理，信号量减1，相当于给add操作加锁
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    [arrayM addObject:@(i)];
    // 数据处理完毕，信号量加1，又可以进行下一次处理
    dispatch_semaphore_signal(semaphore);
});

NSLog(@"count: %zd", arrayM.count);
```

执行结果：

上面的代码是添加了信号量控制的，如果没有的话，很可能就会崩溃啦。

![](/img/in-post/gcd_semaphore_1.png)

显而易见，其完全是乱序的往数组内添加元素。在多线程并发的时候就出现冲突。向数组添加元素这个操作显然不能并发执行，常识。

##### 代码示例2

```objc
// 创建一个组
dispatch_group_t group = dispatch_group_create();

// 信号量初始总量为0
dispatch_semaphore_t semaphore = dispatch_semaphore_create(10);

dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
for (int i = 0; i < 100; i++) {
    // 信号量等待使总量-1，开始10-1=9可以继续往下执行
    // 当循环遍历到10次之后，信号总量变为-1，当前线程就卡住不会继续执行了，可就是等待的样子
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    
    // 将一个并发任务关联到group
    dispatch_group_async(group, queue, ^{
        // 前面10个异步任务，打印出当前循环的i值
        NSLog(@"%i", i);
        // 打印之后，前面10个任务休息2秒
        sleep(2);
        
        // 休眠2秒之后，10个异步任务对信号量各自加1
        // 发送一个信号量+1，如果+1后数量为正，即刻又可以开始执行之前等待的操作
        dispatch_semaphore_signal(semaphore);
    });
}

// 等待group相关的所有任务都执行完才往下走 或者使用notify函数也可以
//    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
//    NSLog(@"完成");

dispatch_group_notify(group, queue, ^{
    NSLog(@"完成");
});
```



##### 代码示例3

问题需求：两个异步任务嵌套，如何保证在内部异步的任务先执行，然后再执行外层的任务。

分析，由于串行队列一定是先添加的任务先执行完毕，所以不行。使用并发队列的话，可以利用信号量达到要求。思路就是，卡主外层的任务，让里层的任务执行完了让信号量加1，然后外层的任务再继续执行。代码如下：

```objc
dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_CONCURRENT);

dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

dispatch_async(queue, ^{
   dispatch_async(queue, ^{
       sleep(2);
       NSLog(@"嵌套执行的任务2 --- %@", [NSThread currentThread]);
       // 让信号量加1
       dispatch_semaphore_signal(semaphore);
   });
    // 信号量等待就会让信号量减1
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"外层的任务1 --- %@", [NSThread currentThread]);
});
```

执行结果：

```objc
嵌套执行的任务2 --- <NSThread: 0x600000070a80>{number = 3, name = (null)}
外层的任务1 --- <NSThread: 0x610000070500>{number = 4, name = (null)}
```

需要注意是，嵌套任务释放信号量一定要等到自己的任务都处理完才释放，不然可能外面的任务就提前或许信号量就执行了。

信号量在网络请求中也有很多应用。

下面以网络请求为例来说明，这个很常见，面试也经常会问到。需要熟悉掌握。

在开发中，我们经常会碰到这样情形，同时发起两个请求，需要等到两个请求都获取到数据之后再进行下一步的操作。但是网络请求是异步的。这个时候，就不要使用到信号量了。

##### 不使用信号量同步

```objc
- (void)dataWithoutSemaphore
{
    self.manager = [AFHTTPSessionManager manager];
    
    NSString *url1 = @"http://120.25.226.186:32812/login";
    NSString *url2 = @"http://120.25.226.186:32812/video";
    
    NSDictionary *params1 = @{
                              @"username" : @"520it",
                              @"pwd" : @"520it"
                              };
    NSDictionary *params2 = @{};
    
    
    // 创建任务组
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    // 将第一个网络请求任务添加到任务组中
    dispatch_group_async(group, globalQueue, ^{
        // 开始网络请求任务
        [self.manager GET:url1 parameters:params1 progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"1 success: %@", [responseObject class]);
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
            NSLog(@"1 failure");
        }];
    });
    
    // 将第二个网络请求任务添加到任务组
    dispatch_group_async(group, globalQueue, ^{
        // 开始网络请求
        [self.manager GET:url2 parameters:params2 progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"2 success: %@", [responseObject class]);
            
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
            NSLog(@"2 failure");
        }];
    });
    
    dispatch_group_notify(group, globalQueue, ^{
        NSLog(@"完成了网络请求，不管是成功还是失败");
    });
}
```

执行结果：

```objc
完成了网络请求，不管是成功还是失败
1 success: __NSSingleEntryDictionaryI
2 success: __NSSingleEntryDictionaryI
```

这是什么意思呢？

说明的是两个网络请求都发送了。结果是什么样子的，不管。框架已经火速把两个异步网络请求发出。下面两行则是对两个网络请求的处理。

在实际的开发中，我们经常需要的是，请求需要异步发出。但是要等到两个请求都得到结果，不管成功还是失败。总要有结果。这个时候，就可以使用信号量来控制线程的同步了。在上面的代码中插入下面的有关线程同步相关的语句。

##### 使用semaphore信号量同步

```objc
- (void)dataWithSemaphore
{
    self.manager = [AFHTTPSessionManager manager];
    
    NSString *url1 = @"http://120.25.226.186:32812/login";
    NSString *url2 = @"http://120.25.226.186:32812/video";
    
    NSDictionary *params1 = @{
                             @"username" : @"520it",
                             @"pwd" : @"520it"
                             };
    NSDictionary *params2 = @{};
    
    
    // 创建任务组
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    // 将第一个网络请求任务添加到任务组中
    dispatch_group_async(group, globalQueue, ^{
        // 创建信号量
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        // 开始网络请求任务
        [self.manager GET:url1 parameters:params1 progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"1 success: %@", [responseObject class]);
            // 如果请求成功，发送信号量
            dispatch_semaphore_signal(semaphore);
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
            NSLog(@"1 failure");
            // 即使请求失败也发送信号量
            dispatch_semaphore_signal(semaphore);
        }];
        
        // 在网络请求任务成功之前，信号量等待中
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    });
    
    // 将第二个网络请求任务添加到任务组
    dispatch_group_async(group, globalQueue, ^{
       // 创建信号量
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        // 开始网络请求
        [self.manager GET:url2 parameters:params2 progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"2 success: %@", [responseObject class]);
            // 如果请求成功，发送信号量
            dispatch_semaphore_signal(semaphore);
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
            NSLog(@"2 failure");
            // 即使请求失败也发送信号量
            dispatch_semaphore_signal(semaphore);
        }];
        // 在网络请求任务成功之前，信号量等待中
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    });
    
    dispatch_group_notify(group, globalQueue, ^{
        NSLog(@"完成了网络请求，不管是成功还是失败");
    });   
}
```

执行结果：

```objc
2 success: __NSSingleEntryDictionaryI
1 success: __NSSingleEntryDictionaryI
完成了网络请求，不管是成功还是失败
```

结果说明：

在异步请求开始的时候，创建一个信号量，等到有处理结果的时候，释放信号量。否则一直等待。请求处理成功或者失败，都会释放信号量。如果不释放信号量，就会一直等待下去。线程等不到结果返回，誓不罢休。group里面的这个任务就相当于没有完成。这样可以满足我们实际开发中很多需求。



### 总结：

发起`网络请求`和`处理响应函数结束`是不一样的。异步发起，瞬间就完事了。但是返回结果，并对结果处理可能非常耗时。也是我们开发中常见的一种情形。通过上面的例子，我们可以看出，`网络请求`的任务提交给子线程异步处理了，网络请求这样的任务也就快速执行完毕了，但是！！！`网络请求`是一个任务，`处理收到的网络响应`又是另外一个任务，注意，在这里AFN请求中前者在异步线程，后者在主线程。一定不要把这两种混为一谈。

信号量的使用前提，想清楚需要哪个线程等待，又要哪个线程继续执行，然后使用信号量。在上面的例子中，block回调是在主线程中执行的，而GET请求是在自己创建的子线程中执行的。所以按照需求，就需要自己创建的异步子线程等待主线程中的block执行完了之后再执行。相当于卡主子线程，主线程没处理完不算完。所以异步子线程需要信号量wait等待，主线程就signal发送信号量。





参考链接：

[http://www.cnblogs.com/goodboy-heyang/p/5277910.html](http://www.cnblogs.com/goodboy-heyang/p/5277910.html)

[http://www.cnblogs.com/goodboy-heyang/p/5271513.html](http://www.cnblogs.com/goodboy-heyang/p/5271513.html)