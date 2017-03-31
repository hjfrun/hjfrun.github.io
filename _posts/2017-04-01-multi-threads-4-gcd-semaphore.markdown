---
layout:     post
title:      iOS多线程（4）
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

### 前言

多线程编程中很重要的一个操作是线程同步。使用GCD进行多线程开发经常会碰到线程同步的情况，下面以网络请求为例来说明，这个很常见，面试也经常会问到。需要熟悉掌握。

在开发中，我们经常会碰到这样情形，同时发起两个请求，需要等到两个请求都获取到数据之后再进行下一步的操作。但是网络请求是异步的。这个时候，就不要使用到信号量了。



#### 不使用信号量同步

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

#### 使用semaphore信号量同步

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

