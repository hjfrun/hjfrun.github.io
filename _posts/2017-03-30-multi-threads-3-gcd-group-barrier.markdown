---
layout:     post
title:      iOS多线程（3）
subtitle:   dispatch_group, 栅栏函数
date:       2017-03-31 21:03:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - 多线程
    - GCD
---



##### 1、异步开启子线程做一些耗时操作（下载，耗时图片处理等），操作完成，回到主线程更新UI

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  // 异步开启子线程下载一张图片
    NSData *imageData = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9.jpg"]];
    
    UIImage *image = [UIImage imageWithData:imageData];
    dispatch_async(dispatch_get_main_queue(), ^{
      // 在主线程更新UI
        self.imageView.image = image;
    });
});

```



##### 2、`dispatch group`的使用

要完成多个任务，彼此间不用关心先后顺序。如果只是使用上面最基本的dispatch_async则会是这样：

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  // image1 下载完再开始下载image2
    UIImage *image1 = [self imageWithURLString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1490976633211&di=73abb81787b984c953dc9f33d0637cb9&imgtype=0&src=http%3A%2F%2Ffile06.16sucai.com%2F2016%2F0921%2Fda78bbfe5a27798a8d300f30d5ad594e.jpg"];
    UIImage *image2 = [self imageWithURLString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1490976633210&di=0f9b4c445a6be4566f2864f180ad7777&imgtype=0&src=http%3A%2F%2Fe.hiphotos.baidu.com%2Fzhidao%2Fpic%2Fitem%2Fb3b7d0a20cf431ad5fd0ae584d36acaf2edd9855.jpg"];
    
    dispatch_async(dispatch_get_main_queue(), ^{
        self.imageView.image = image1;
        self.imageView2.image = image2;
    });
});

// 根据传入的图片URL地址下载图片
- (UIImage *)imageWithURLString:(NSString *)urlString
{
    NSData *imageData = [NSData dataWithContentsOfURL:[NSURL URLWithString:urlString]];
    return [UIImage imageWithData:imageData];
}
```

这个时候，image2会等待image1下载完成才会开始下载。很多时候，我们不关心他们谁先下载完成，可以让它们一起开始下载。提高下载的效率。但是如果仅仅是这样，直接开两个异步线程去下载就可以了。但是我经常是碰到让两个任务同时开始执行，然后都执行完之后，再去执行一个任务三。

这个时候，`dispatch_group`就派上用场了。

```objc
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_CONCURRENT);

// 异步下载图片
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    
    __block UIImage *image1 = nil;
    __block UIImage *image2 = nil;
    
  	// 关联一个任务到group
    dispatch_group_async(group, queue, ^{
       	// 下载第一张图片
        image1 = [self imageWithURLString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1490976633211&di=73abb81787b984c953dc9f33d0637cb9&imgtype=0&src=http%3A%2F%2Ffile06.16sucai.com%2F2016%2F0921%2Fda78bbfe5a27798a8d300f30d5ad594e.jpg"];
        
    });
    // 关联一个任务到group
    dispatch_group_async(group, queue, ^{
      	// 下载第二张图片
        image2 = [self imageWithURLString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1490976633210&di=0f9b4c445a6be4566f2864f180ad7777&imgtype=0&src=http%3A%2F%2Fe.hiphotos.baidu.com%2Fzhidao%2Fpic%2Fitem%2Fb3b7d0a20cf431ad5fd0ae584d36acaf2edd9855.jpg"];
    });
    
  	// 等待组中的任务执行完毕，回到主线程执行block
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        self.imageView.image = image1;
        self.imageView2.image = image2;
    });
    
});
```

`dispatch_group_notify`函数用来指定一个额外的block，该block将在group中所有任务执行完之后再执行。

###### `dispatch_group_wait`函数

函数原型：`dispatch_group_wait(dispatch_group_t group,  dispatch_time_t timeout)`

前面参数指定要等待的组。后面一个时间表示超时时间。一旦调用这个wait函数，该函数就处理调用的状态而不返回值，只有当函数的currentThread停止，或到达wait函数指定的等待的时间，或者dispatch_group中的操作全部执行完毕，执行该函数的线程停止。

当timeout为`DISPATCH_TIME_FOREVER`就意味着永久等待。当timeout为`DISPATCH_TIME_NOW`就意味不用任何等待，即可判定group中的任务是否全部执行结束。如果`dispatch_group_wait`函数的返回值不为0，就意味着虽然经过了指定的时间，但是dispatch_group中的任务并未全部执行完毕。如果`dispatch_group_wait`函数的返回值为0。就意味着dispatch_group中的任务都已经执行完毕。

使用`dispatch_group_wait`的代码示例：

```objc
// 创建一个组
dispatch_group_t group = dispatch_group_create();

// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 封装异步任务到任务组
dispatch_group_async(group, queue, ^{
    NSLog(@"第一张图片开始下载");
    sleep(2);
    NSLog(@"第一张图片下载完成");
});

dispatch_group_async(group, queue, ^{
    NSLog(@"第二张图片开始下载");
    sleep(2);
    NSLog(@"第二张图片下载完成");
});

dispatch_group_async(group, queue, ^{
    NSLog(@"第三张图片开始下载");
    sleep(2);
    NSLog(@"第三张图片下载完成");
});

dispatch_group_async(group, queue, ^{
    NSLog(@"第四张图片开始下载");
    sleep(2);
    NSLog(@"第四张图片下载完成");
});

// 使用wait函数表示主要group中有操作没有结束，就一直等待
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"图片都下载好了，开始更新UI");
```

运行结果：

```objc
12:31:44.116 GCDDemo[62202:3220385] 第一张图片开始下载
12:31:44.116 GCDDemo[62202:3220397] 第二张图片开始下载
12:31:44.116 GCDDemo[62202:3220383] 第三张图片开始下载
12:31:44.116 GCDDemo[62202:3220382] 第四张图片开始下载
12:31:46.120 GCDDemo[62202:3220385] 第一张图片下载完成
12:31:46.120 GCDDemo[62202:3220383] 第三张图片下载完成
12:31:46.120 GCDDemo[62202:3220397] 第二张图片下载完成
12:31:46.120 GCDDemo[62202:3220382] 第四张图片下载完成
12:31:46.121 GCDDemo[62202:3220334] 图片都下载好了，开始更新UI
```



###### 使用dispatch_group_enter和dispatch_group_leave实现组处理

在函数开始时调用enter，在函数结束后调用leave。

代码示例：

```objc
// 创建一个组
dispatch_group_t group = dispatch_group_create();

// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 队列的任务进入组
dispatch_group_enter(group);
dispatch_async(queue, ^{
    NSLog(@"第一张图片开始下载");
    sleep(2);
    NSLog(@"第一张图片下载完成");
    // 队列的任务执行完毕，离开组
    dispatch_group_leave(group);
});

dispatch_group_enter(group);
dispatch_async(queue, ^{
    NSLog(@"第二张图片开始下载");
    sleep(2);
    NSLog(@"第二张图片下载完成");
    dispatch_group_leave(group);
});

dispatch_group_enter(group);
dispatch_async(queue, ^{
    NSLog(@"第三张图片开始下载");
    sleep(2);
    NSLog(@"第三张图片下载完成");
    dispatch_group_leave(group);
});

dispatch_group_enter(group);
dispatch_async(queue, ^{
    NSLog(@"第四张图片开始下载");
    sleep(2);
    NSLog(@"第四张图片下载完成");
    dispatch_group_leave(group);
});


// 使用wait函数表示主要group中有操作没有结束，就一直等待
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"图片都下载好了，开始更新UI");
```

执行结果：

```objc
12:40:51.953 GCDDemo[62229:3233750] 第一张图片开始下载
12:40:51.953 GCDDemo[62229:3233748] 第二张图片开始下载
12:40:51.953 GCDDemo[62229:3233762] 第三张图片开始下载
12:40:51.953 GCDDemo[62229:3233747] 第四张图片开始下载
12:40:53.957 GCDDemo[62229:3233762] 第三张图片下载完成
12:40:53.957 GCDDemo[62229:3233747] 第四张图片下载完成
12:40:53.957 GCDDemo[62229:3233748] 第二张图片下载完成
12:40:53.957 GCDDemo[62229:3233750] 第一张图片下载完成
12:40:53.958 GCDDemo[62229:3233672] 图片都下载好了，开始更新UI
```

效果跟上面的结果类似。太直观了，无法解释。



##### 3、栅栏函数dispatch_barrier_async的使用

顾名思义，在线程管理中充当了一个栅栏的作用。我们有5个任务。1、2并发执行，完毕之后3执行，3执行完之后，4、5再并发执行，这个时候适合用到这个栅栏函数。可以实现高效率的数据库访问和文件访问。实例：

```objc
// 注意这里一定要使用自己生成的并发队列，而不能是获取的系统提供的并发队列
dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(queue, ^{
    NSLog(@"block 1: %@", [NSThread currentThread]);
});

dispatch_async(queue, ^{
    NSLog(@"block 2: %@", [NSThread currentThread]);
});

dispatch_barrier_async(queue, ^{
    NSLog(@"barrier : %@", [NSThread currentThread]);
});

dispatch_async(queue, ^{
    NSLog(@"block 4: %@", [NSThread currentThread]);
});

dispatch_async(queue, ^{
    NSLog(@"block 5: %@", [NSThread currentThread]);
});
```

执行结果：

```objc
block 2: <NSThread: 0x610000267c80>{number = 4, name = (null)}
block 1: <NSThread: 0x6080002643c0>{number = 3, name = (null)}
barrier : <NSThread: 0x6080002643c0>{number = 3, name = (null)}
block 4: <NSThread: 0x6080002643c0>{number = 3, name = (null)}
block 5: <NSThread: 0x610000267c80>{number = 4, name = (null)}
```

如果将上面栅栏函数改为sync执行，则结果如下：

```objc
block 1: <NSThread: 0x60000007c0c0>{number = 3, name = (null)}
block 2: <NSThread: 0x61000007df00>{number = 4, name = (null)}
barrier : <NSThread: 0x6100000787c0>{number = 1, name = main}
block 4: <NSThread: 0x61000007df00>{number = 4, name = (null)}
block 5: <NSThread: 0x60000007c0c0>{number = 3, name = (null)}
```

仍然能够起到栅栏的作用，需要注意的是这个的barrier任务是在主线程执行的，1、2，4、5看起来和它们的添加顺序一致，实际只是巧合。多运行几次就不同了。

特别注意这里的并发队列一定是自己创建的，不能是全局并发队列或者是串行队列。`dispatch_barrier_async`的任务在异步线程执行，`dispatch_barrier_sync`在主线程执行。`dispatch_barrier_sync`需要等待自己的任务执行完之后才会继续程序。





