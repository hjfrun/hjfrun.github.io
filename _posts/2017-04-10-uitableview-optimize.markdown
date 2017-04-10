---
layout:     post
title:      UITableView优化
subtitle:   
date:       2017-04-10 11:07:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - iOS
    - UITableView
---

UITableView在我们的开发中非常常用。其在使用过程中，经常会碰到滑动卡顿的问题。在这里总结下它的优化策略。

### Cell重用

`Cell重用` 这个是优化的第一步。首先创建一个静态变量`reuseID`，然后从缓冲池中取相应的identifier的cell并更新数据，如果没有才会去创建新的`cell`。并用`identifier`标识`cell`。每个`cell`都会注册一个`identifier`（`可重用标识符`）放入缓存池，当需要调用的时候就直接从缓存池里找到对应的`id`。当不需要的时候，就放入缓存池里待用。移除屏幕的`cell`才会放入缓存池，并不会`release`。

### 定义尽量少的Cell 善用hidden subview

分析`cell`的结构。尽可能将相同的内容抽取到一种样式的`cell`中，保证`UITableView`要显示多少内容，真正创建出的`cell`可能只比屏幕显示的`cell`多一点。虽然`cell`的体积可能会大点，但是因为`cell`的数量不会很多，完全可以接受。好处是减少代码量，减少`nib`文件的数量。统一一个`nib`文件定义`cell`，容易修改和维护。

只定义一种`cell`，该如何显示不同类型的内容呢？答案是，把所有的不同类型的`view`都定义好，放在`cell`里，通过`hidden`显示、隐藏，来显示不同类型的内容。毕竟，在用户快速滑动中，只是单纯的显示、隐藏`subview`比实时的创建要快得多

### 提前计算并缓存cell的高度

在`iOS`开发中，不设`UITableView`的预估行高的情况下，会频繁调用`tableView:heightForRowAtIndexPath:`方法获取即将显示的`cell`的高度，可以在这里进行优化。如果已经计算过了，就把结果缓存起来，不用每次都重头计算。

### 异步绘制

遇到比较复杂的界面时，如复杂的图文混排，

### 滑动时，按需加载

网络不好的时候，异步加载图片是个正常程序员都会想到的。思想是，在快速滑动过程中，只加载目标范围内的`cell`，这样按需加载，极大的提高李长度。`SDWebImage`可以实现异步加载，尤其是在现实大量图片展示的时候。

```objc
//获取可见部分的Cell
NSArray *visiblePaths = [self.tableView indexPathsForVisibleRows];
for (NSIndexPath *indexPath in visiblePaths)
{
//获取的dataSource里面的对象，并且判断加载完成的不需要再次异步加载
     <code>
}

// tableView 停止滑动的时候异步加载图片
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath

if (self.tableView.dragging == NO && self.tableView.decelerating == NO)
{
   //开始异步加载图片
    <code>
}

```

### 避免大量的图片缩放、颜色渐变等，尽量显示“大小刚好合适的图片资源”

### 避免同步的从网络、文件获取数据，cell内实现的内容来自web，使用异步加载，缓存请求结果

### 渲染

子控件的层级越深，渲染到屏幕上所需要的计算量就越大，多用`drawRect:`绘制元素，替代用`view`显示；

对于不透明的`view`，设置`opaque`为`YES`，这样在绘制该`view`时，就不需要考虑被`view`覆盖的其他内容，尽量设置`opaque`为`YES`。避免`GPU`对`cell`下面的内容也进行绘制；

避免`CALayer`特效

```objc
view.layer.shadowColor = color.CGColor;
view.layer.shadowOffset = offset;
view.layer.shadowOpacity = 1;
view.layer.shadowRadius = radius;
```



总结，`tableview`的优化主要是以下几个方向：

* 提前计算高度并缓存；
* 按需加载，防止卡顿，配合`SDWebImage`；
* 异步绘制
* 缓存一切可以缓存的。




参考：

[http://blog.csdn.net/u011452278/article/details/60961350](http://blog.csdn.net/u011452278/article/details/60961350)

