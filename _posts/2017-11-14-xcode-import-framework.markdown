---
layout:     post
title:      如何在Xcode多项目方便引用framework
subtitle:   
date:       2017-11-14 09:10:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - Xcode
---

我们在workspace中有两个项目，A和B。这两个project有一些共用的组件，工具或者UI。希望把这两部分抽到一个Core（核心）项目。然后A和B分别引用Core。

打个比方。

我们需要一个条件打log的小工具。在测试环境可以输出调试log到控制台。但是到了线上release的时候，移除这些log。封装了一个工具类。把这个类拖到Core项目。这个时候，只要在A和B中import都可以打印log。



这个时候又有问题了。虽然我们可以在需要打log的地方import这个Core。但是项目里面需要打log的地方实在太多了。几乎每个文件都需要打印log。难道每个文件我们都要把这个Core导入一次吗？其实大可不必。解决方案是在桥接文件中一次导入：**@import Core**即可。



当然了，在实际中肯能比这个还要复杂。因为Core可能有OC文件，可能有Swift分类，OC分类。Core还依赖于Pods。