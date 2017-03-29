---
layout:     post
title:      给UITextView添加占位文字
subtitle:   
date:       2017-03-29 11:07:35
author:     "hjfrun"

header-img: ""
catalog: false
tags:
    - iOS
    - Foundation
---



Foundation框架里面UITextField是有占位文字的属性，而UITextView默认是没有占位文字的，去头文件里也是找不到的。但是给UITextView增加占位文字是一个很常见的需求。我也碰到过。梳理下之前自己的一个简单的实现。

还记得用户在发送iMessage的时候，那个输入框是一个带有占位文字的TextView，我们可以猜想，是不是系统提供的UITextView是带有一个内部属性就是表示占位文字的。这个简单，使用runtime挖掘一下UITextView的实例变量就知道了。

```objc
unsigned int output = 0;
Ivar *ivars = class_copyIvarList([UITextView class], &output);

for (NSInteger i = 0; i < output; i++) {
    Ivar ivar = ivars[i];
    const char *ivarName = ivar_getName(ivar);
    const char *ivarType = ivar_getTypeEncoding(ivar);
    
    NSLog(@"%s, %s", ivarName, ivarType);
}
    
free(ivars);
```



执行效果：

![](/img/in-post/textview-placeholder-1.png)

可以看得出来还真有一个内部的_placeholderLabel。我们只需要把这个使用起来就可以了。

使用方法：

```objc
UILabel *label = [[UILabel alloc] init];
label.text = @"占位文字";
label.textColor = [UIColor redColor];
label.font = [UIFont systemFontOfSize:13.f];
    
[self.textView addSubview:label];
[self.textView setValue:label forKey:@"_placeholderLabel"];
```

这样就配置了我们自己的占位label给textView了。是不是很简单，哈哈。



这是一开始想到的方法。后来发现，还有更简单的方法或许可以查看到一个类的内部变量。

调试的时候打断点就可以了。

![](/img/in-post/textview-placeholder-2.png)



需要说明的是，这种方法只在iOS8之后的系统有效。iOS7和之前的，还是要自己添加，然后监听文本通知等传统繁琐的方法。这里就不写了。别人的实现已经太多了。

