---
layout:     post
title:      iOS多线程（7）【NSOperation篇】
subtitle:   基础用法 
date:       2017-04-03 00:54:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - NSOperation
    - 多线程
---

iOS最新的多线程技术是NSOperation & NSOperationQueue。我们在GCD中已经了解过任务和队列的概念了。正好就对应了这里的NSOperation和NSOperationQueue。NSOperation是在Cocoa框架下基于GCD的封装，相对于GCD来说可控性更强。并且加入了一些方便功能。



#### 什么是NSOperation

NSOperation翻译成中文的意思就是任务或者说操作。它是一个抽象类。不能直接实例化对象使用。需要使用它的具体之类的对象。系统已经提供了两个它的子类NSInvocationOperation和NSBlockOperation。前者要结合self-selector方式使用，后者是直接把任务封装成block使用，类似GCD中的任务。需要了解的是在Swift中，前者被认为在多线程下是不安全的，貌似已经废弃了。如果系统提供的这两种具体子类不能满足我们的需求，还可以根据需要进行子类化定制。前面我们学GCD的时候知道，任务都是放到队列queue中去执行的。*但是在NSOperation不必要，可以调用其start方法使其执行。*但是我们在实际开发中一般不会这么做，用NSOperation就是为了多线程，就是为了到队列中去并发执行。话不多说，先来几段代码跑跑看结果。

##### **NSInvocationOperation**的创建使用：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    [self op1];
}

- (void)op1
{
    // 创建NSInvocationOperation对象，test为调用方法。
    NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(test) object:nil];
    [invocationOperation start];
}

- (void)test
{
    NSLog(@"test: %@", [NSThread currentThread]);
}

// 执行结果
test: <NSThread: 0x61800006d6c0>{number = 1, name = main}
```

##### **NSBlockOperation**的创建和使用

```objc
NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"created block: %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"add block 1: %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"add block 2: %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"add block 3: %@", [NSThread currentThread]);
}];

[blockOperation addExecutionBlock:^{
    NSLog(@"add block 4: %@", [NSThread currentThread]);
}];

[blockOperation start];
```

执行结果：

```objc
created block: <NSThread: 0x610000066c40>{number = 1, name = main}
add block 3: <NSThread: 0x600000065800>{number = 5, name = (null)}
add block 4: <NSThread: 0x61000006ba80>{number = 6, name = (null)}
add block 2: <NSThread: 0x61800006b800>{number = 4, name = (null)}
add block 1: <NSThread: 0x608000066b00>{number = 3, name = (null)}
```

可以看得出来，创建时候配置的block是在主线程执行的，而后面通过对象方法add进去的block是并发执行的。并发的一般是在子线程完成，`但是也有可能在主线程执行`！！！

`注意`NSBlockOperation的任务一旦开始，是不能再往里面添加任务的。在上面的例子中，如果把后面的start方法写在前面的任务添加之前，则会报错。

```objc
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[NSBlockOperation addExecutionBlock:]: blocks cannot be added after the operation has started executing or finished'

```

##### 依赖关系

NSOperation还提供了一个非常实用的方法，添加依赖。这是GCD所没有的。通过设置依赖，我们可以设置不同任务间的依赖关系，确定他们之间的执行顺序。注意任务间不要添加循环依赖，会导致死锁。

还要`注意`的是添加依赖要在任务并发执行的时候，如果任务是在串行队列执行，则添加依赖是没有什么卵用的。下面的代码充分说明了这种情况，注意执行时间。

```objc
- (void)op3
{
    NSBlockOperation *downloadOperation1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"begin download image 1");
        sleep(2);
        NSLog(@"download finished image 1");
        NSLog(@"block thread 1 : %@", [NSThread currentThread]);
    }];
    
    NSBlockOperation *downloadOperation2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"begin download image 2");
        sleep(2);
        NSLog(@"download finished image 2");
        NSLog(@"block thread 2 : %@", [NSThread currentThread]);
    }];
    
    NSBlockOperation *editOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"edit image1 & image2");
        NSLog(@"block thread edit : %@", [NSThread currentThread]);
    }];
    
    // 设置依赖
    [editOperation addDependency:downloadOperation1];
    [editOperation addDependency:downloadOperation2];
    
    [downloadOperation1 start];
    [downloadOperation2 start];
    [editOperation start];
}
```

运行结果：

```objc
00:28:26.389 NSOperationDemo[71311:4294353] begin download image 1
00:28:28.390 NSOperationDemo[71311:4294353] download finished image 1
00:28:28.391 NSOperationDemo[71311:4294353] block thread 1 : <NSThread: 0x608000073e40>{number = 1, name = main}
00:28:28.391 NSOperationDemo[71311:4294353] begin download image 2
00:28:30.393 NSOperationDemo[71311:4294353] download finished image 2
00:28:30.393 NSOperationDemo[71311:4294353] block thread 2 : <NSThread: 0x608000073e40>{number = 1, name = main}
00:28:30.394 NSOperationDemo[71311:4294353] edit image1 & image2
00:28:30.394 NSOperationDemo[71311:4294353] block thread edit : <NSThread: 0x608000073e40>{number = 1, name = main}
```

##### completionBlock

当任务执行完毕时，将会调用一次这个block，方便我们更新UI或者添加其他的业务逻辑。

##### addObserver

可以通过添加观察者监听任务的执行情况。像普通的对象KVO一样。



#### NSOperationQueue

操作队列。和GCD中的任务队列是一回事。

##### 基本用法

```objc
// 创建操作队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
// 设置maxConcurrentOperationCount
queue.maxConcurrentOperationCount = 1;

// 添加具体任务
[queue addOperation:operation];
// 直接添加block任务到队列
[queue addOperationWithBlock:^{
    // 任务
}];
```

设置最大并发数，默认是-1，表示不限制数量，由系统运行期环境决定。设为1就相当于串行队列。

`[NSOperationQueue mainQueue]`获得当前的主队列。

##### 其他用法

```objc
// 队列内所有任务的数组
@property (readonly, copy) NSArray<__kindof NSOperation *> *operations;
// 队列内的任务数
@property (readonly) NSUInteger operationCount;
// 是否暂停队列(这个方法不会导致正在执行的任务中途暂停,只是简单地阻断新任务的执行)
@property (getter=isSuspended) BOOL suspended;
// 取消队列中所有的任务
- (void)cancelAllOperations;
// 阻塞当前线程直到队列中的所有任务执行完毕
- (void)waitUntilAllOperationsAreFinished;

```

任务一旦添加到queue中。就会自动执行，不需要再调用任务的- start方法。

以后有时间再来研究下自定义`NSOperation`



更进一步的学习：[http://www.cnblogs.com/YouXianMing/p/3707403.html](http://www.cnblogs.com/YouXianMing/p/3707403.html)