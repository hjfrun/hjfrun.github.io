---
layout:     post
title:      iOS应用优化的建议及技巧
subtitle:   
date:       2017-04-20 21:14:35
author:     hjfrun
header-img: 
catalog: true
tags:
    - iOS
    - 优化
---

在开发一个`iOS`应用的时候，一个非常重要的方面是应用的性能。你的用户需要你的应用有良好的性能表现，如果应用反应迟钝，会得到糟糕的用户反馈。

然而，由于`iOS`设备的限制，很多时候这并不容易。但还是有一些需要开发者谨记的技巧，可以帮助开发者避免一些性能的影响。

> 在优化之前，确定有一个问题需要被解决。不要卷入过早优化的错误。经常使用Instruments来静态检查你的代码和检查其他需要优化的部分。

这些优化主要分为3个等级：

## 入门技巧：

这一部分致力于使用一些基础改变来提升应用的性能。但是所有级别的开发者都可以从这些checklist中获益良多。

### 1. 使用ARC来管理内存

`ARC`是`Automatic Reference Counting`的缩写。诞生于iOS 5。大大解决了一些常见的内存泄露问题。它通过自动管理retain/release。

`ARC`除了避免内存泄露之外，还能优化应用性能，原因是它能确保对象在其不再使用的时候能够立即释放。如果哪家iOS开发团队还没使用ARC管理项目，尽早离开这种傻逼项目组。

> 需要注意的是，ARC并不能解决所有的内存泄露问题。不恰当的`block`使用，循环引用，错误的`CoreFoundation`对象的管理等都会导致内存泄露。



### 2. 在合适的地方使用可重用标识符

一个常见的低级错误是没有正确的给`UITableViewCells`、`UICollectionViewCells`及其他HeaderFooterViews设置可重用标识符。

使用`tableView`的一个良好实践是，`tableview`的数据源需要重复使用Cell对象。`tableview`持有一个队列或者链表标记为可重用的`cells`。

如果不使用可重用标识符。每次需要展示一个`cells`的时候，都需要重新创建一个`cell`。这是一个代价非常大的操作，极大的影响应用的滑动性能。

自iOS 6之后，最好是给`header`和`footer view`使用可重用标识符。`UICollectionView` 的cells和`supplementart views`也是一样。

```objc
static NSString *CellIdentifier = @"Cell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier forIndexPath:indexPath];
```

这么做可以从队列中取出一个可用的`cell`，或者创建一个新的`cell`。注意，这个`cell`需要已经使用`nib`或者`class`注册，否则返回`nil`。



### 3. 设置view的opaque属性为YES

设置`view`为不透明，`iOS`系统在绘制这个`view`的时候，不会去渲染`view`后面的视图。尽量使用`opaque`为`YES`的视图。可以通过`xib`或者代码设置。

> This property provides a hint to the drawing system as to how it should treat the view. If set to YES, the drawing system treats the view as fully opaque, which allows the drawing system to optimize some drawing operations and improve performance. If set to NO, the drawing system composites the view normally with other content. The default value of this property is YES.

对于一个相对静态的页面来说，不设置这个`opaque`属性为`YES`或许不会有太大的影响，但是如果`view`是嵌套在`scrollview`里面、或者是一个复杂动画的一部分。这就会在很大程度上影响程序的性能。

可以使用模拟器的`Debug/Color Blended Layers`选项来查看哪些`view`没有设置为`opaque`。我们的目标是尽可能多的设置`view`的`opaque`为`YES`。



### 4. 避免Fat XIBs

这个fat可以理解为沉重的，负担重的。`storyborad`在iOS 5之后迅速取代xib。但是xib在很多地方仍然使用非常广泛。

如果你不能避免的要使用xib的话，尽量保证使用的xib尽量简单。

还有需要注意的是，xib在加载的时候，这个xib里面的所有内容都是一次性直接加载到内存中。包括图片。如果一个view不是立即使用的，这样会浪费宝贵的内存。在storyboard中是不会存在这种情况的。

当使用xib的时候，所有图片都会缓存起来。在OS X系统中，音频文件都会缓存起来。

> When you load a nib file that contains references to image or sound resources, the nib-loading code reads the actual image or sound file into memory and and caches it. In OS X, image and sound resources are stored in named caches so that you can access them later if needed. In iOS, only image resources are stored in named caches. To access images, you use the imageNamed: method of NSImage or UIImage, depending on your platform.



### 5. 不要阻塞主线程

永远不要在主线程中执行耗时重的任务。因为UIKit的所有任务都在主线程完成，渲染、触摸反馈，输入响应。

把所有操作都在主线程执行的风险是，这样会阻塞主线程，这样程序就会显得卡顿。

大部分阻塞主线程的情况都发生在大量`IO操作`的时候，需要读取或者写入外设大量对象。磁盘或者网络。

可以使用`异步线程`来规避这些情况。

其他的，如果是执行一些昂贵操作，可以使用`多线程`技术。`GCD`或者`NSOperationQueue`

使用`GCD`的一个典型模板如下：

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // switch to a background thread and perform your expensive operation
 
    dispatch_async(dispatch_get_main_queue(), ^{
        // switch back to the main thread to update your UI
 
    });
});
```



### 6. 调整图片的尺寸到跟ImageView大小一致

手动调整图片的尺寸到跟`imageView`大小一致，至少的比例一致。避免一些像素压缩，对齐的消耗。



### 7. 使用合适的集合类

* `Arrays`：数组，有序的一列对象。可以根据索引快速遍历，插入、删除操作慢，代价大；
* `Dictionaries`：字典，存储键值对，按键查找快，底层使用红黑树实现；
* `Sets`：集合，无序值对象，按值查找快，可以很快的插入和删除；



### 8. 使用GZIP压缩

使用`GZIP`压缩，在传输大量内容的时候，可以使用`GZIP`压缩处理，减轻传输负担。



## 中级技巧：

### 9. 重复使用以及懒加载views

更多的`views`意味着更多的渲染，最终意味着更多的`CPU`计算以及内存消耗。当嵌入`views`到`UIScorllView`的时候体现更明显。

一个常见的优化技巧是，不要一次性把这些子视图都创建好，而是等到需要的时候再去创建，不需要的时候，添加可重用队列。

这样，在滑动操作执行的时候，只需要去配置这个view的数据，避免了昂贵的资源分配操作。

举个栗子，比如，当你点击一个按钮，然后展现一个view的情况，可以使用以下两种方法来实现：

1. 当这个页面一出现的时候，就去创建这个view，并且hide起来，当你需要的时候，在show出来；
2. 在需要显示前什么都不做，当需要显示的时候，才去创建和显示，一次完成；

这两种方法，各自都有优缺点，是个人都想得明白，不细述了。



### 10. 缓存

一条黄金准则是，缓存一切重要。一些重要的，不会轻易改变的，使用频繁的对象。都可以缓存起来。

哪些需要缓存起来的呢？网络请求的响应，图片，甚至的计算出来的值，例如`tableview`的高度。

`NSURLConnection`根据`HTTP`请求头来决定缓存策略，内存缓存或者磁盘缓存。你还可以手动配置一个`NSURLRequest`使其只加载缓存对象。

下面是一个很好的`snippet`，当你需要一个`NSURLRequest`来加载一个不太会变的图片情形：

```objc
+ (NSMutableURLRequest *)imageRequestWithURL:(NSURL *)url {
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
 
    request.cachePolicy = NSURLRequestReturnCacheDataElseLoad; // this will make sure the request always returns the cached image
    request.HTTPShouldHandleCookies = NO;
    request.HTTPShouldUsePipelining = YES;
    [request addValue:@"image/*" forHTTPHeaderField:@"Accept"];
 
    return request;
}
```

如果是需要缓存`HTTP`请求无关的对象时，可以使用`NSCache`。

`NSCache`看起来使用起来像字典，当系统需要回收内存时，它能够移除它的内容。



### 11. 渲染

[Designing for iOS: Graphics & Performance](https://robots.thoughtbot.com/designing-for-ios-graphics-performance) 



### 12. 处理内存警告

iOS会通知所有应用，当可用内存不足的时候。

```objc
If your app receives this warning, it must free up as much memory as possible. The best way to do this is to remove strong references to caches, image objects, and other data objects that can be recreated later.
```

幸运的是，`UIKit`提供几种方式来处理低内存通知。

* 在代理方法里实现`applicationDidReceiveMemoryWarning:`方法；
* 重载控制器的`didReceiveMemoryWarning`方法；
* 监听`UIApplicationDidReceiveMemoryWarningNotification`通知

一旦接受到内存警告，响应方法里面，就需要立即释放不需要的内存。

例如，接收到内存警告通知时，控制器会释放其view，当这个view是不可见的时候。如果你的应用在接受到内存警告时，不过内存释放处理，你的应用则很可能被系统杀死。

需要注意的是，当你在有选择的释放一些对象的时候，需要确保这些对象在后面是可以再被重新创建的。可以使用模拟器来模拟内存警告。

### 13. 重用昂贵对象

有一些对象的创建很慢，代价很大。`NSDataFormatter` & `NSCalendar`是两个例子。但是你总避免不了使用它们。当需要处理大量这样的对象的时候，可以重复使用它们，而不要每次都创建。给你的类添加一个`属性`，强持有这些对象，或者使用`静态变量`。

注意，如果你使用的是第二种方法，这个对象会一直存在当你的程序一直运行的时候，有点像单例对象。

下面的代码示范了，懒加载一个属性的情况。

```objc
// in your .h or inside a class extension
@property (nonatomic, strong) NSDateFormatter *formatter;
 
// inside the implementation (.m)
// When you need, just use self.formatter
- (NSDateFormatter *)formatter {
    if (! _formatter) {
        _formatter = [[NSDateFormatter alloc] init];
        _formatter.dateFormat = @"EEE MMM dd HH:mm:ss Z yyyy"; // twitter date format
    }
    return _formatter;
}
```

上面的代码不是线程安全的。如果要做到线程安全，可以使用下面的实现：

```objc
// no property is required anymore. The following code goes inside the implementation (.m)
 
- (NSDateFormatter *)formatter {
    static NSDateFormatter *formatter;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _formatter = [[NSDateFormatter alloc] init];
        _formatter.dateFormat = @"EEE MMM dd HH:mm:ss Z yyyy"; // twitter date format
    });
    return formatter;
}
```

需要注意的是，设置一个`NSDateFormatter`的`date format`和创建一个几乎一样慢，因此，如果你的应用要频繁的使用到不同的`date formats`，不如一次性创建好这些不同的对象。



### 14. 避免重复处理数据

许多应用需要从远程服务器请求信息，这些数据可能从`JSON`或者`XML`中解析而来，所以很重要的是重复使用这些已经解析好的数据结构，当这些结构并没有被改变的时候。

为什么呢？因为在内存中处理这些数据结构的代价是昂贵的。

举例，你需要在`tableview`中展示大量的数据，最好是把这些请求的结果放到一个数组里面。使用的时候，不要去直接改变这些对象，否则后面又需要每次都重新解析。大概是这个意思。

通过恰当的方式存储这些解析的数据，可以避免后面很多重复的数据处理操作。



### 15. 选择合适的数据格式

前端和后台数据交换的方式有很多种，最常见的方式是`JSON`和`XML`。你需要确保在你的应用内选用合适的方式。

`JSON`解析起来更快，而且一般来说，都比`XML`要小。这些意味着更小的数据传输。自`iOS 5`之后内置的`JSON`序列化类使用起来更加方便。

和`JSON`的解析方式不一样的是，它有`SAX`解析，可以边解析边操作。不需要等到全部数据都下载好并且一次性加入到内存来解析。这种方式可以节省资源，提升性能。



### 16. 恰当的设置背景图片

给一个`view`设置背景图片的方式常用的有两种：

* 设置背景的`UIColor`通过`colorWithPatternImage`方法；
* 给`view`添加一个子`UIImageView`；

第一种方法是，图片以模式不断重复，覆盖到`view`的背景。第二种适合添加一张全尺寸的背景图。这其实没什么好说的。。。



### 17. 设置Shadow Path

当你需要给一个`view`或者一个`layer`设置阴影的时候，一般的实践是使用`QuartzCore`框架的方法，代码可能如下：

```objc
#import <QuartzCore/QuartzCore.h>
 
// Somewhere later ...
UIView *view = [[UIView alloc] init];
 
// Setup the shadow ...
view.layer.shadowOffset = CGSizeMake(-1.0f, 1.0f);
view.layer.shadowRadius = 5.0f;
view.layer.shadowOpacity = 0.6;
```

坏消息是，使用这种方法，`Core Animation`会执行一个离屏渲染，来准确的计算`view`的形状。在这个`view`的阴影。这个操作是非常昂贵的。

好消息是，使用`shadow path`是一个很好的实践：

```objc
view.layer.shadowPath = [[UIBezierPath bezierPathWithRect:view.bounds] CGPath];
```

通过设置`shadow path`，iOS系统不用每次都重复计算阴影。替代的，它会使用你提前告知的，已经预计算好的路径。



### 18. 优化tableView

我们都知道`tableview` 需要快速流畅的滑动，如若不然，是极其影响用户体验的部分。其实在本文的第一部分就已经涉及到部分。可重用`cells`。总结来说，要保持`tableview`的流畅滑动，需要遵从以下的这些建议：

* 设定可重用标识符的可重用`cells`
* 尽量让`views`为`opaque`，包括`cell`本身
* 避免`gradients`、图片压缩以及离屏渲染
* 如果每行高度不一样，缓存每行cell的高度
* 如果cell展示的内容，有的是来自网络，要确保这些请求是异步的，而且要缓存请求回来的结果
* 在用到阴影的时候时候`shaodow path`
* 尽量减少子`view`数量，`subview`
* 在`cellForRowAtIndexPath：`方法尽量少执行任务，原因是这个方法执行很多次，应该尽量快的返回，如果你需要做些什么，尽量只做一次，而且把结果缓存起来
* 使用合适的数据结构来存储那些你需要的信息，不同的结构对不同的操作有不同的代价
* 尽量使用`rowHeight`、`sectionFooterHeigh`t和`sectionHeaderHeight`来设置高度，而避免去问代理



### 19. 选择合适的数据持久化方案

常见的数据持久化方案，包括：

* 使用偏好设置`NSUserDefaults`
* 保存到结构化存储的文件，例如`XML`、`JSON`或者`plist`格式的文件
* 使用`NSCoding`协议归档
* 保存到本地数据库例如`SQLite`
* 使用`Core Data`

说明：

1. 偏好设置使用起来方便，但是只使用与存储一些非常小量的数据，主要是一些应用配置之类的数据；
2. 使用第二种方案，保存到其他结构化的文件，需要把整个文件一次性读入到内存，然后解析这是一个代价非常昂贵的方案。你也可以使用`SAX`方式来处理`XML`文件，但仍然是一个复杂的解决方案；
3. `NSCoding`使用起来跟上面的方法一样麻烦，如果是对自定义的对象进行归档，需要这些对象实现归档协议。其他方面跟上面两种方法是一致的；
4. 最佳实践方案是使用`SQLite`或者`Core Data`。使用数据库，你不需要一次性把所有的内容读入到内存。性能方面这两种方案差不多。使用数据库的优点多多。`SQLite`是一种常见的`DBMS`，数据库管理系统。然而`Core Data`表示的是对象的图形化模型；如果你选择使用`SQLite`，那么使用`FMDB`是一个不错的方案。



## 高级技巧

### 20. 加速启动

加速启动最重要的要诀是，尽可能的把任务放到异步线程去执行，例如`网络请求`，`数据库访问`，`数据库解析`等等；

同样的，需要避免重量化的xibs。



### 21. 使用自动释放池 Autorelease Pool

`自动释放池`是在一个block中自动释放不再需要使用的对象。一般来说，由UIKit自动调用，但是在有的情况下需要程序员手动的来创建自动释放池。

例如，如果你创建一堆临时对象的时候。你需要这些临时对象在不需要使用的时候立即释放，不然会占据大量的内存空间，但是，如果你不自己创建自己的自动释放池。而是等到`UIKit`来抽空自动释放池。那么这些内存对象会存活更长的时间而得不到释放。

现在使用了自动释放池，可以使用`@autoreleasepool`把这段产生大量临时对象的代码包裹起来，离开这段代码，会自动释放这些代码中生成的临时对象。代码示例如下：

```objc
NSArray *urls = <# An array of file URLs #>;
for (NSURL *url in urls) {
    @autoreleasepool {
        NSError *error;
        NSString *fileContents = [NSString stringWithContentsOfURL:url
                                         encoding:NSUTF8StringEncoding error:&error];
        /* Process the string, creating and autoreleasing more objects. */
    }
}
```



### 22. 图片缓存

从`app bundle`中加载图片有两种方式，最常见的方式是使用`imageNamed`方法，第二种是`imageWithContentsOfFile`。这两种方法有什么区别呢。`imageNamed`方法会缓存图片

> This method looks in the system caches for an image object with the specified name and returns that object if it exists. If a matching image object is not already in the cache, this method loads the image data from the specified file, caches it, and then returns the resulting object.

相反的，`imageWithContentsOfFile`只是简单的加载图片没有缓存。

```objc
UIImage *img = [UIImage imageNamed:@"myImage"]; // caching
// or
UIImage *img = [UIImage imageWithContentsOfFile:@"myImage"]; // no caching
```

那么该如何决策呢，当使用的图片是使用一次，那么就不需要缓存。在这种情况下，`imageWithContentsOfFile`是比较合适的。系统不会浪费空间来缓存这个图片。

然而，当图片会缓存，适合于那些比较小，使用多次的图片。这种情况下，使用`imageNamed`方法更合适。操作系统节省了时间，不用每次都从磁盘加载这些图片内容。

