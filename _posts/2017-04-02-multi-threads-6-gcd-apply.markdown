---
layout:     post
title:      iOS多线程（6）
subtitle:   apply 
date:       2017-04-02 13:12:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - GCD
---



`dispatch_apply`该函数按指定的次数将指定的block追加到指定的队列中，**`并等待全部执行结束`**这些block的执行并发执行，但是一定要等到全部执行完毕才可以进行下一步。

代码示例1：

最简单的apply。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_apply(10, queue, ^(size_t index) {
    NSLog(@"%zd", index);
});

NSLog(@"完成");
```

运行结果：

```objc
 1
 0
 3
 2
 4
 5
 6
 7
 8
 9
 完成
```



代码示例2：

在block内访问到外部数据，在block中对外部的数据进行处理。注意，这里对数据的处理操作是有要求的。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

NSArray *array = @[@"a", @"b", @"c", @"d", @"e", @"f", @"g", @"h", @"i", @"j"];

dispatch_apply(array.count, queue, ^(size_t index) {
    NSLog(@"%zd: %@", index, array[index]);
});

NSLog(@"完成");
```

运行结果：

```objc
1: b
0: a
4: e
2: c
3: d
7: h
5: f
8: i
6: g
9: j
完成
```

说明：

最后的打印【完成】一定会在apply中的任务全部执行完毕。

代码示例3：

在子线程批量做大量的事情，完成之后，再回到主线程更新UI。

```objc
// 获取全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_async(queue, ^{
   
    dispatch_apply(10, queue, ^(size_t index) {
        NSLog(@"开始下载图片: %zd", index);
        sleep(2);
        NSLog(@"下载图片完毕: %zd", index);
    });
    
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"图片都下载好了，开始更新UI");
    });
});
```

运行结果：

```objc
13:10:16.403 GCDDemo[63606:3271833] 开始下载图片: 0
13:10:16.403 GCDDemo[63606:3271834] 开始下载图片: 2
13:10:16.403 GCDDemo[63606:3271852] 开始下载图片: 1
13:10:16.403 GCDDemo[63606:3271855] 开始下载图片: 5
13:10:16.403 GCDDemo[63606:3271853] 开始下载图片: 3
13:10:16.403 GCDDemo[63606:3271836] 开始下载图片: 4
13:10:16.403 GCDDemo[63606:3271856] 开始下载图片: 6
13:10:16.403 GCDDemo[63606:3271857] 开始下载图片: 7
13:10:18.408 GCDDemo[63606:3271855] 下载图片完毕: 5
13:10:18.408 GCDDemo[63606:3271833] 下载图片完毕: 0
13:10:18.408 GCDDemo[63606:3271836] 下载图片完毕: 4
13:10:18.408 GCDDemo[63606:3271853] 下载图片完毕: 3
13:10:18.408 GCDDemo[63606:3271852] 下载图片完毕: 1
13:10:18.408 GCDDemo[63606:3271834] 下载图片完毕: 2
13:10:18.408 GCDDemo[63606:3271856] 下载图片完毕: 6
13:10:18.408 GCDDemo[63606:3271857] 下载图片完毕: 7
13:10:18.408 GCDDemo[63606:3271855] 开始下载图片: 8
13:10:18.409 GCDDemo[63606:3271833] 开始下载图片: 9
13:10:20.412 GCDDemo[63606:3271855] 下载图片完毕: 8
13:10:20.412 GCDDemo[63606:3271833] 下载图片完毕: 9
13:10:20.413 GCDDemo[63606:3271751] 图片都下载好了，开始更新UI
```



其他的，前面讲利用信号量实现线程同步的时候，也代码示例了有关apply实现的。也可以参考下:[GCD利用信号量实现线程同步](http://hjfrun.com/2017/04/01/multi-threads-4-gcd-semaphore/)