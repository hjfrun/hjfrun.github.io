---
layout:     post
title:      iOS多线程（上）
subtitle:   pthread & NSThread简介
date:       2017-03-30 18:58:35
author:     "hjfrun"
header-img: ""
catalog: true
tags:
    - iOS
    - 多线程
---



iOS开发中有四种多线程实现方案。分别是：

* `pthread`
* `NSThread`
* `GCD`
* `NSOperation & NSOperationQueue`

在这里打算用三篇文章把这里面的四种实现方案都简单的梳理下，以及我在实际工作中的运用。记录以备后查。

这一篇就简单的说下前面两种用得不多的，但是对原理需要知道点的。



##### `pthread`

在类`Unix`操作系统中，都是使用`pthread`作为操作系统的线程。基于C语言实现，可移植性强。但是用起来麻烦。使用要包含头文件`#import <pthread.h>`

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    pthread_t thread;
  	// 创建一个线程会自动执行
    pthread_create(&thread, NULL, printCurrentThread, NULL);
}

void * printCurrentThread(void *data)
{
    NSLog(@"%@", [NSThread currentThread]);
    return NULL;
}
```

全部用C语言函数来实现，需要程序员手动来处理线程的各个状态转换，即管理生命周期。上面创建了线程，但是并没有销毁。



##### `NSThread`

既然是NS开头的，就知道是苹果在`pthread`基础上进行了面向对象的封装。可以直接操控线程对象，但是依然需要手动管理，偶尔用用，就像上面那个地方，调试的时候很方便打印当前线程的信息。和`pthread`不一样的是，创建之后，可以控制启动方式。

* 先创建，再启动

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(threadAction:) object:nil];
}

- (void)threadAction:(NSThread *)thread
{
    NSLog(@"currentThread: %@", [NSThread currentThread]);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    [self.thread start];
}
```



* 创建并自动执行

```objc
[NSThread detachNewThreadWithBlock:^{
        NSLog(@"thread with block");
    }];
    
    [NSThread detachNewThreadSelector:@selector(threadAction:) toTarget:self withObject:nil];
```

以上两种方式，第一种是把任务直接放到block里面，一种是把任务放到selector里面执行。两种方式创建的线程都是一经创建，立即执行。



* 使用`NSObject`的方法创建并且自动启动

```objc
[self performSelectorInBackground:@selector(threadAction:) withObject:nil];
```



* 其他注意事项：

`NSThread`是OC封装的，提供了- cancel, -start, -main方法。能设置优先级。使用perform方法还能任务在哪个线程执行，并指定runloop运行模式：

```objc
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
	// equivalent to the first method with kCFRunLoopCommonModes

- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array NS_AVAILABLE(10_5, 2_0);
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait NS_AVAILABLE(10_5, 2_0); // equivalent to the first method with kCFRunLoopCommonModes
```

还有一些方法获取线程的执行状态：

```objc
@property (readonly, getter=isExecuting) BOOL executing NS_AVAILABLE(10_5, 2_0);
@property (readonly, getter=isFinished) BOOL finished NS_AVAILABLE(10_5, 2_0);
@property (readonly, getter=isCancelled) BOOL cancelled NS_AVAILABLE(10_5, 2_0);
```

线程睡眠：

```objc
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)ti;
```

其他非常用方法，参考头文件`NSThread.h`
