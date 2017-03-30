---
layout:     post
title:      iOS多线程（2)
subtitle:   GCD相关
date:       2017-03-30 20:06:35
author:     "hjfrun"

header-img: ""
catalog: true
tags:
    - iOS
    - 多线程
    - GCD
---



**GCD**全称`Grand Central Dispatch`，是苹果为多核并行计算提出的解决方案。会自动合理地利用更多的CPU内核，而且会自动管理线程的生命周期，包括创建、调度、销毁。完全不需要我们管理。只需要布置任务，告诉它去做什么就行。这也说明了缺乏一些对线程的操作。同时也是C语言的，使用了block，用起来方便灵活，常用。



在`GCD`中有两个非常重要的概念：**任务**和**队列**。

- 任务：即操作，想要做什么，说白了就是一段代码，在GCD中就是一个block，所以添加任务方便。任务有两种执行方式：**同步执行**和**异步执行**。
  * 同步执行：同步（sync）不具备开启新线程的能力，等待操作完成才返回；
  * 异步执行：异步（async）具备开启新线程的能力，直接返回，不用等待操作完成；
- 队列：用户存放任务，一共有两种队列，**串行队列**和**并行队列**。
  * 串行队列：`serial_queue`**串行执行**队列中的任务会先进先出FIFO的执行；
  * 并行队列：`concurrent_queue`**并发执行**放到并行队列中的任务，GCD也会FIFO的取出来，但不同的是，它取出来一个就会放到一个线程，然后再取一个任务，可能放到另外一个线程。这样由于取的动作很快，看起来，所有的任务都是一起执行的。GCD会根据系统资源控制并发数量，如果任务很多，它并不会让所有任务同时执行。

结论，dispatch_sync和dispatch_async都是用于提交任务，它们的区别是会不会等待任务完成。对于dispatch_sync而言，它提交任务，同时阻塞当前的线程，直到任务执行完成才会返回。dispatch_async会在提交任务之后直接返回，不阻塞等待。如此一来，真正与线程相关的就是队列了。

所有组合的在一起：

![](/img/in-post/gcd_sketch.png)



对应上面6种情形，分情况实现以下：

* 1、 异步执行 + 并行队列

  * 实现代码：

    ```objc
    - (void)asyncConcurrent
    {
        // 创建一个并行队列
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

* 2、异步执行+串行队列

  * 实现代码：

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


* 3、同步执行+并行队列
  * 实现代码：
  
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

* 4、同步执行+串行队列
  * 实现代码：
  
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
	
* 5、异步执行+主队列
  * 实现代码：
	
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

* 6、同步执行+主队列
  * 实现代码：
  
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





参考文章：

[http://www.cocoachina.com/ios/20161205/18281.html](http://www.cocoachina.com/ios/20161205/18281.html)

[http://www.cocoachina.com/ios/20161031/17887.html](http://www.cocoachina.com/ios/20161031/17887.html)
