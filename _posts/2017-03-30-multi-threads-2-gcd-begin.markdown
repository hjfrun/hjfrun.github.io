---
layout:     post
title:      iOS多线程（2）【GCD篇】
subtitle:   GCD基础
date:       2017-03-30 20:06:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - 多线程
    - GCD
---



## `GCD`简介

**GCD**全称`Grand Central Dispatch`，是苹果为多核计算提出的解决方案。会自动合理地利用更多的CPU内核，而且会自动管理线程的生命周期，包括创建、调度、销毁。完全不需要我们管理。只需要布置任务，告诉它去做什么就行。这也说明了缺乏一些对线程的操作。同时也是C语言的，使用了block，用起来方便灵活，常用。在`GCD`中有两个非常重要的概念：**任务**和**队列**。


### `调度队列`

1. 调度队列的核心理念是，将长期运行的任务拆分为多个工作单元，并将这些单元添加到`dispatch_queue`中，系统为我们管理这些`dispatch_queue`，为我们在多个线程上执行工作单元，我们不需要直接启动和管理后台线程。
2. 系统提供了许多预定义的`dispatch_queue`，包括可保证在主线程上执行的`dispatch_queue`也可以创建自己的`dispatch_queue`，可以创建任意多个。GCD的`dispatch_queue`严格遵循`FIFO`原则，添加到`dispatch_queue`的工作单元始终按照加入顺序启动。
3. `dispatch_queue`按FIFO的顺序，串行或并发地执行任务。
  * `serial_queue`一次只能执行一个任务，当前任务完成才能开始出列并启动下一个任务；
  * `concurrent_queue`则尽可能多地启动任务并发执行。

### 创建和管理`dispatch_queue`

1. 获得全局并发队列（`concurrent_queue`）
  * 并发队列可以同时并发执行多个任务，不过仍然按照FIFO顺序来启动任务。并发queue会在之前一个任务完成之前就出列下一个任务并开始执行。并发queue并发执行的任务数量会根据应用和系统动态变化。各种因素包括：可用核数量、其他进程正在执行的工作数量、其他串行dispatch_queue中优先任务的数量等。
  * 系统给每个应用提供四个dispatch_queue，整个应用内部全局共享，四个queue的区别是优先级。可以使用dispatch_get_global_queue函数来获取这四个queue：

    ```objc
    dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    #define DISPATCH_QUEUE_PRIORITY_HIGH 2
    #define DISPATCH_QUEUE_PRIORITY_DEFAULT 0
    #define DISPATCH_QUEUE_PRIORITY_LOW (-2)
    #define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
    ```
    第一个参数用于指定优先级，第二个参数目前未使用到，默认0即可。
  * 虽然dispatch_queue是引用计数的对象，但不需要retain和release全局并发queue。因为这些queue对应用是全局的，retain和release调用会忽略。也不需要存储这三个queue的引用，每次直接调用dispatch_get_global_queue获取queue即可。

2. 创建串行队列（`serial_queue`）
  * 应用的任务需要按照特定的顺序执行，就需要使用串行队列，串行队列每次只执行一个任务。可以使用queue来代替锁，保护共享资源或者可变的数据结构。可锁不一样，串行queue确保任务按可预测的顺序执行。而且只要是异步提交任务到串行queue，就永远不会产生死锁。

### 添加任务到queue

要执行一个任务，需要将它添加到一个合适的`dispatch_queue`中，可以单个或者按组来添加，也可以同步或者异步的去执行一个任务。一旦进入到queue，queue会尽快的执行任务。一般任务用block封装。
1. 添加单个任务到queue
  * 异步添加任务
    异步（async）具备开启新线程的能力，不用等待操作完成，直接返回；异步添加任务用`dispatch_async`或者`dispatch_async_f`函数异步地调度任务。因为添加任务到queue中，无法确定这些代码什么时候能够执行。不用等待这些任务执行完成，就可以去做其他事情。
  * 同步添加任务
    同步（sync）不具备开启新线程的能力，等待操作完成才返回；少数时候可能希望同步地调度任务，以免竞争条件或者其他同步错误。使用`dispatch_sync`和`dispatch_sync_f`函数同步地添加任务到queue。这两个函数会阻塞当前线程，知道相应任务完成执行。注意：绝对不要在任务中调用`dispatch_sync`或`dispatch_sync_f`函数，并同步调度新任务到当前正在执行的queue。因为这样做会导致死锁；而并发队列也应该避免这样做。


结论，`sync`和`async`都是用于提交任务，它们的区别是会不会等待任务完成。对于`sync`而言，它提交任务，同时阻塞当前的线程，直到任务执行完成才会返回。`async`会在提交任务之后直接返回，不阻塞等待。

所有组合的在一起：

![](/img/in-post/gcd_sketch.png)



对应上面6种情形，分情况实现以下：

##### 1、 异步执行 + 并发队列

*   实现代码：

    ```objc
    - (void)asyncConcurrent
    {
        // 创建一个并发队列
        dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_CONCURRENT);
        
        NSLog(@"---start---");
        
        // 使用异步函数封装3个任务
        dispatch_async(queue, ^{
            NSLog(@"block 1 --- %@", [NSThread currentThread]);
        });
        
        dispatch_async(queue, ^{
            NSLog(@"block 2 --- %@", [NSThread currentThread]);
        });
        
        dispatch_async(queue, ^{
            NSLog(@"block 3 --- %@", [NSThread currentThread]);
        });
        
        NSLog(@"---end---");
        
    }
    ```

* 打印结果：

    ```objc
    ---start---
    ---end---
    block 2 --- <NSThread: 0x608000078180>{number = 4, name = (null)}
    block 1 --- <NSThread: 0x610000074e00>{number = 3, name = (null)}
    block 3 --- <NSThread: 0x6000000762c0>{number = 5, name = (null)}
    ```

* 说明：

    * 异步执行意味着：
      * 可以开启新的线程
      * 任务可以先绕过不执行，回头再来执行
    * 并发队列意味着：
      * 任务之间不需要排队，不保证执行的先后顺序
    * 两者组合后的结果：
      * 开启了3个不同的新线程
      * 函数在执行时，先打印了start和end，再回头执行这5个任务
      * 这三个任务是并发执行的，先后顺序和添加的顺序不保证一致。执行是2->1->3。 

* 步骤图：

  ![](/img/in-post/gcd_thread_1.png)

##### 2、异步执行+串行队列

*   实现代码：

    ```objc
    - (void)asyncSerial
    {
        // 创建一个串行队列
        dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_SERIAL);
        
        NSLog(@"---start---");
        
        // 使用异步函数封装5个任务
        dispatch_async(queue, ^{
            NSLog(@"block 1 --- %@", [NSThread currentThread]);
        });
        
        dispatch_async(queue, ^{
            NSLog(@"block 2 --- %@", [NSThread currentThread]);
        });
        
        dispatch_async(queue, ^{
            NSLog(@"block 3 --- %@", [NSThread currentThread]);
        });
        
        NSLog(@"---end---");
    }
    ```

* 执行结果

    ```objc
    ---start---
    ---end---
    block 1 --- <NSThread: 0x608000069480>{number = 3, name = (null)}
    block 2 --- <NSThread: 0x608000069480>{number = 3, name = (null)}
    block 3 --- <NSThread: 0x608000069480>{number = 3, name = (null)}
    ```

* 解释

 * 异步执行意味着：
   - 可以开启新的线程
   - 任务可以绕过不执行，回头再来执行
 * 串行队列意味着：
   - 任务必须按照添加的先后顺序执行
 * 两者组合后的结果：
   - 开启了一个新的子线程
   - 函数在执行时，先打印了start和end，在回头执行这3个任务
   - 这三个任务是按顺序执行的，所以结果是1->2->3
 * 步骤图：

 ![](/img/in-post/gcd_thread_2.png)

##### 3、同步执行+队列

*   实现代码：

   ```objc
   	- (void)syncConcurrent
   	{
   	    // 创建一个并发队列
   	    dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_CONCURRENT);

   	    NSLog(@"---start---");

   	    // 使用同步函数封装3个任务
   	    dispatch_sync(queue, ^{
   		NSLog(@"block 1 --- %@", [NSThread currentThread]);
   	    });

   	    dispatch_sync(queue, ^{
   		NSLog(@"block 2 --- %@", [NSThread currentThread]);
   	    });

   	    dispatch_sync(queue, ^{
   		NSLog(@"block 3 --- %@", [NSThread currentThread]);
   	    });

   	    NSLog(@"---end---");
   	}
   ```
* 结果：

   ```
   	---start---
   	block 1 --- <NSThread: 0x60000007ef00>{number = 1, name = main}
   	block 2 --- <NSThread: 0x60000007ef00>{number = 1, name = main}
   	block 3 --- <NSThread: 0x60000007ef00>{number = 1, name = main}
   	---end---
   ```
* 解释：
 - 同步执行意味着：
  * 不能开启新的线程
  * 任务创建后必须执行完成才能往下走
 - 并发队列意味着：
  * 任务必须按照添加队列顺序挨个执行
 - 两者组合意味着：
  * 所有任务都只能在主线程中执行
  * 函数在执行时，必须按照代码中的书写顺序一行一行的执行完才能继续
 - 注意事项：
  * 在这里即便是并发队列，任务可以并发执行，但是由于只存在一个主线程，所以没法把任务分发到不同的线程去同步执行，其结果就是只能在主线程里按顺序挨个执行。
 * 步骤图：

 ![](/img/in-post/gcd_thread_3.png)

##### 4、同步执行+串行队列

*   实现代码：

    ```objc
    - (void)syncSerial
    {
        // 创建一个串行队列
        dispatch_queue_t queue = dispatch_queue_create("com.hjfrun", DISPATCH_QUEUE_SERIAL);
        
        NSLog(@"---start---");
        
        // 使用同步函数封装3个任务
        dispatch_sync(queue, ^{
            NSLog(@"block 1 --- %@", [NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            NSLog(@"block 2 --- %@", [NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            NSLog(@"block 3 --- %@", [NSThread currentThread]);
        });
        
        NSLog(@"---end---");
    }
    ```
* 结果：

    ```objc
    ---start---
    block 1 --- <NSThread: 0x60000006ad80>{number = 1, name = main}
    block 2 --- <NSThread: 0x60000006ad80>{number = 1, name = main}
    block 3 --- <NSThread: 0x60000006ad80>{number = 1, name = main}
    ---end---
    ```
* 解释：
    这里执行的原理和步骤和`同步执行+并发队列`一样，只是同步执行没法开启新的线程，所以多个任务之间也是一样只能按顺序来执行

##### 5、异步执行+主队列

*   实现代码：

    ```objc
    - (void)asyncMain
    {
    	// 获取主队列
    	dispatch_queue_t queue = dispatch_get_main_queue();

    	NSLog(@"---start---");

    	// 使用异步函数封装3个任务
    	dispatch_async(queue, ^{
    	    NSLog(@"block 1 --- %@", [NSThread currentThread]);
    	});

    	dispatch_async(queue, ^{
    	    NSLog(@"block 2 --- %@", [NSThread currentThread]);
    	});

    	dispatch_async(queue, ^{
    	    NSLog(@"block 3 --- %@", [NSThread currentThread]);
    	});

    	NSLog(@"---end---");
    }
    ```
* 结果：

    ```objc
    ---start---
    ---end---
    block 1 --- <NSThread: 0x618000067640>{number = 1, name = main}
    block 2 --- <NSThread: 0x618000067640>{number = 1, name = main}
    block 3 --- <NSThread: 0x618000067640>{number = 1, name = main}
    ```
* 解释：
    - 异步执行意味着：
      * 可以开启新的线程
      * 任务可以先绕过不执行，回头再来执行
    - 主队列跟串行队列的区别：
      * 队列中的任务一样要顺序执行
      * 主队列中的任务必须在主线程中执行，不允许在子线程中执行
    - 以上条件组合后得出结果：
      * 所有任务都可以先跳过，之后再按顺序执行
* 步骤图：

  ![](/img/in-post/gcd_thread_5.png)

##### 6、同步执行+主队列

*   实现代码：

    ```objc
    - (void)syncMain
    {
        // 获取主队列
        dispatch_queue_t queue = dispatch_get_main_queue();
        
        NSLog(@"---start---");
        
        // 使用同步函数封装3个任务
        dispatch_sync(queue, ^{
            NSLog(@"block 1 --- %@", [NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            NSLog(@"block 2 --- %@", [NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            NSLog(@"block 3 --- %@", [NSThread currentThread]);
        });
        
        NSLog(@"---end---");
    }
    ```
* 结果：

    ```objc
    ---start---
    ```
* 解释：
    - 主队列中的任务必须按顺序挨个执行
    - 任务1要等主线程有空的时候（即主队列中所有任务执行完）才能执行
    - 主线程要执行完打印`---end---`的任务后才有空
    - 任务1和打印`---end---`两个任务相互等待，造成死锁
* 步骤图：

  ![](/img/in-post/gcd_thread_6.png)


### 在iOS开发中的基础应用



参考文章：

[http://www.cocoachina.com/ios/20161205/18281.html](http://www.cocoachina.com/ios/20161205/18281.html)

[http://www.cocoachina.com/ios/20161031/17887.html](http://www.cocoachina.com/ios/20161031/17887.html)
