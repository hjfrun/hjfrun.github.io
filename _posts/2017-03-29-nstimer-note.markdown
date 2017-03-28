---
layout:     post
title:      NSTimer相关知识点
subtitle:   ""
date:       2017-03-29 14:09:35
author:     "hjfrun"
header-img: ""
catalog: false
tags:
    - iOS
    - 学习
---





**NSTimer**定时器是```Foundation```框架提供的常用对象，也是iOS开发中最常用到的定时器。当然了，iOS开发中还有别的定时器，比如**CADisplayLink**和**GCD**定时器。后两种在这里暂不做讨论，今天梳理下最常用的**NSTimer**定时器。



#### 基本用法

```objective-c
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
```

使用注意：

我们知道NSTimer需要添加到run loop中才可以正常工作的，使用第一个方法创建的定时器需要手动添加到定时器，不然不会执行，在添加的时候可以选择需要添加到的模式。使用第二个方法创建的定时器已经添加到主run loop 的default模式。

上面两种方法都是任务放到selector中，时间到了再去执行。至iOS 10之后，又添加了两个新方法，把任务封装到了block中去。使用起来更加方便。

```objective-c
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));
```

