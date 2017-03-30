---
layout:     post
title:      iOS的内存管理相关
subtitle:   
date:       2017-03-30 14:12:35
author:     "hjfrun"

header-img: ""
catalog: true
tags:
    - iOS
    - ARC
    - 内存管理
---



iOS现在的内存管理使用的是自动引用计数（Automatic Reference Count 简称ARC），这种技术早已有之。苹果在2011年将这种技术运用于iOS的内存管理，在那个我还没有进入iOS开发的远古时代，iOS的程序员需要手动来管理内存，叫做MRC（Manual Reference Count）。管理内存需要手写很多内存管理相关的语句。retail、release、autorelease等等。现在ARC的实现已经非常成熟了，程序员应该积极迎接这项技术。苹果很大程度上解放了程序员的一些繁琐的工作。虽说如此，程序员还是应该对内存管理相关的技术原理有一些基本的理解。不过如果出去找工作，聊天起来，如果哪家公司还说他们的工程使用的是MRC。碰到这种垃圾公司，果断弃坑比较好。



##### 什么是ARC

​	引用技术是一个非常简单而有效的管理对象生命周期的方式。当我们创建一个新对象的时候，它的引用计数为当有一个新的指针指向这个对象时，我们将其引用计数加1，当某个指针不再指向这个对象的时候，我们将其引用计数减1，当对象的引用计数变为0时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。出了Objective-C语言外。其他语言也有基于自动引用计数的内存管理方式。

​	ARC相对于MRC，不是在编译时添加retain、release、autorelease这么简单，是在编译期和运行期两部分帮助开发者管理内存。在编译期，ARC用的是更底层的C接口实现的retain、release、autorelease，这样性能更好，也是为什么不能在ARC环境下手动添加这些语句。ARC也包含运行期组件，这个地方优化较为复杂。但也不能被忽略，这部分苹果的编译期已经优化得够好了。不需要开发者参与。



##### 什么是GC

垃圾回收技术。Garbage Collection。是另外一种典型的垃圾回收技术，和引用技术从实现机理上有很大的不同。Java语言就是最典型的运用这种技术的语言。详细的后续再专门写文章讨论。下面简单的记录下这两个的不同以及优缺点。



##### ARC相对于GC的优点

* ARC工作在编译期，在运行时没有额外开销；
* ARC的内存回收是平稳进行的，对象不被使用时基本上是立即被回收。而GC的内存回收是一阵一阵的，回收时需要暂停程序，会有一定的卡顿。

##### ARC相对于GC的缺点

* GC简单，基本上完全不用处理内存管理的问题，而ARC还是需要处理类似的循环引用这种内存管理的问题。
* GC一类的语言学起来更简单。如Java。




##### iOS开发中常见的循环引用的场景

* 定时器（NSTimer）：NSTimer经常会被当做某个类的成员变量，而NSTimer初始化时要制定self为target，容易造成循环应用（self->timer->self)。另外，如果timer一直处于有效validate状态，则其引用计数始终大于0，因此，在不再使用定时器以后，应该先调用invalidate方法使定时器失效释放target引用。有关NSTimer相关讨论可以参考这篇文章[NSTimer简单梳理](http://hjfrun.com/2017/03/28/nstimer-note/)。
* block的使用：block在copy时都会对block内部用到的对象进行强引用（ARC）或者retainCount增加1（MRC）。在ARC和非ARC环境下对block使用不当都会引起循环引用问题。一般表现为，某个类将block作为自己的属性变量，然后该类在block的方法体里面又使用了该类本身，简单的说就是self.someBlock = ^{[self dosomething]};或者self.otherVar = xxx;或者__var = …;出现循环的原因是：self->block->self或者self->block->\_ivar（成员变量）。
* 代理（delegate）：这种问题其实很初级了。已经没什么可说的了。如果连这个都不知道怎么避免，就不用写iOS了。解决办法就是声明delegate时用assign(MRC)或者weak(ARC)。




