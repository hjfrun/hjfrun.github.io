---
layout:     post
title:      iOS多线程（4）
subtitle:   suspend & resume & GCD timer
date:       2017-04-01 22:48:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - GCD
    - 计时器
---



####  `dispatch_suspend & dispatch_resume`

`dispatch_suspend`挂起队列，但是不等于立即停止队列的执行。`dispatch_resume`恢复队列的功能。

对于已经开始的任务，即便是挂起了，也要等待它的执行结束。

```objc
dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_SERIAL);

// 提交第一个block，延迟5秒打印
dispatch_async(queue, ^{
    NSLog(@"开始第一个任务");
    sleep(5);
    NSLog(@"等待5秒之后...第一个任务终于执行完毕啦");
});

// 提交第二个block，延迟5秒打印
dispatch_async(queue, ^{
    NSLog(@"开始第二个任务");
    sleep(5);
    NSLog(@"等待5秒之后...第二个任务终于执行完毕啦");
});

// 延迟1秒
NSLog(@"延迟1秒");
sleep(1);

// 挂起队列
NSLog(@"挂起队列");
dispatch_suspend(queue);

// 延迟10秒
NSLog(@"延迟10秒");
[NSThread sleepForTimeInterval:10];

// 恢复队列
NSLog(@"恢复队列");
dispatch_resume(queue);
```

运行结果：

```objc
23:00:02.806 GCDDemo[58212:2594456] 开始第一个任务
23:00:02.806 GCDDemo[58212:2594379] 延迟1秒
23:00:03.807 GCDDemo[58212:2594379] 挂起队列
23:00:03.808 GCDDemo[58212:2594379] 延迟10秒
23:00:07.812 GCDDemo[58212:2594456] 等待5秒之后...第一个任务终于执行完毕啦
23:00:13.809 GCDDemo[58212:2594379] 恢复队列
23:00:13.809 GCDDemo[58212:2594457] 开始第二个任务
23:00:18.814 GCDDemo[58212:2594457] 等待5秒之后...第二个任务终于执行完毕啦
```

分析上面的执行顺序，观察执行的时间顺序。可以看出，即便已经挂起队列，添加里面的任务也是要执行完成的。如果是并发队列，如果暂停得稍微晚一点，则可能出现所有任务都已经提交到线程中去执行了。



#### `GCD计时器`

示例代码：

```objc
@property (nonatomic, strong) dispatch_source_t timer;

_timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue());
dispatch_source_set_timer(_timer, DISPATCH_TIME_NOW, dispatch_time(DISPATCH_TIME_NOW, 0), 0 * NSEC_PER_SEC);
dispatch_source_set_event_handler(_timer, ^{
    NSLog(@"定时器任务");
});
dispatch_resume(_timer);
```

GCD定时器实质是`runloop`的source的一种，在敲代码dispatch source的时候Xcode会提示你是不是要敲定时器出来。直接补全即可。

![](/img/in-post/gcd_timer_1.png)

![](/img/in-post/gcd_timer_2.png)

注意要让控制器强应用计时器，不然创建完就挂了。block里面的任务不会执行。

