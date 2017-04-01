---
layout:     post
title:      NSTimer简单梳理
subtitle:   
date:       2017-03-28 14:09:35
author:     "hjfrun"
header-img: ""
catalog: false
tags:
    - iOS
    - 学习
    - 计时器
---


**NSTimer**定时器是`Foundation`框架提供的常用对象，也是iOS开发中最常用到的定时器。当然了，iOS开发中还有别的定时器，比如**CADisplayLink**和**GCD**定时器。后两种在这里暂不做讨论，今天梳理下最常用的**NSTimer**定时器。



#### 基本用法

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
```

使用注意：

我们知道NSTimer需要添加到`runloop`中才可以正常工作的，使用第一个方法创建的定时器需要手动添加到定时器，不然不会执行，在添加的时候可以选择需要添加到的模式。使用第二个方法创建的定时器已经添加到主`runloop` 的default模式。

上面两种方法都是任务放到selector中，时间到了再去执行。至iOS 10之后，又添加了两个新方法，把任务封装到了block中去。使用起来更加方便。而且，官方注释写得很清楚，新的API可以规避蛋疼的循环引用问题。但是这个API只在

```objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

/// - parameter:  block  The execution body of the timer; the timer itself is passed as the parameter to this block when executed to aid in avoiding cyclical references
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));
```



```objc
- (void)timerWithBlock
{
    __weak typeof(self) weakSelf = self;
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.f repeats:YES block:^(NSTimer * _Nonnull timer) {
        __strong typeof(self) strongSelf = weakSelf;
        [strongSelf timerTick:timer];
    }];
}

- (void)timerTick:(NSTimer *)timer
{
    NSLog(@"timer tick counts: %zd", ++_second);
    
    self.label.text = [NSString stringWithFormat:@"%zd", _second];
    
    if (_second == 5) {
        [self.timer invalidate];
        self.timer = nil;
    }
}
```



#### iOS 10之前解决循环引用

常见的两种方法：

* 添加NSTimer的分类，在分类内实现一个weakTimer，用block包裹任务，苹果iOS 10新增的API即是仿造的这种方法；
* 使用```MSWeakTimer```

主要说说第一种方法，其实参考系统的实现就明白了。

1、新增分类方法

**NSTimer+WeakTimer.h**

```objc
+ (NSTimer *)weakScheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block;
```

**NSTimer+WeakTimer.m**

```objc
+ (NSTimer *)weakScheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *))block
{
    return [self scheduledTimerWithTimeInterval:interval target:self selector:@selector(blockHandler:) userInfo:[block copy] repeats:repeats];
}

+ (void)blockHandler:(NSTimer *)timer;
{
    void (^block)(NSTimer *timer) = timer.userInfo;
    if (block) {
        block(timer);
    }
}
```

2、使用分类方法

跟上面iOS 10API的一样，这里就不重复了。


我们项目里面都是用的[MSWeakTimer](https://github.com/mindsnacks/MSWeakTimer)，这是一个线程安全的Timer，不会对target进行retain操作，支持GCD Queue，支持Pod引入，使用方便。


#### 其他注意事项

1、NSTimer需要添加到`runloop`才能执行。且要是跑起来的`runloop`。子线程的`runloop`是懒加载的。需要把NSTimer添加在子线程的`runloop`，选择相应的模式，并调用run方法才可以工作；

2、`UIScrollView`环境下如果要NSTimer也工作，要选择tracking模式或者选择common标记的模式。



#### 使用场景

1、在一个页面开启定时器，这个页面不释放，跳转到其他页面。要求在离开这个页面的时候定时器停止工作，再回到这个页面的时候定时器继续工作。使用下面的方法。

```objc
// 页面消失的时候让定时器停止工作
- (void)viewWillDisappear:(BOOL)animated
{
    [self.timer setFireDate:[NSDate distantFuture]];
}
// 页面回来时让定时器继续工作
- (void)viewWillAppear:(BOOL)animated
{
    [self.timer setFireDate:[NSDate distantPast]];
}
```

注意`invalidate`方法是让定时器失效，这个是永久停止。

可以使用`isValid`属性判断定时器是否还有效

```objc
@property (readonly, getter=isValid) BOOL valid;
```



简单的就梳理这么多，后续可能还会完善。。。


参考Demo代码链接[NSTimerDemo](https://github.com/hjfrun/NSTimerDemo)
