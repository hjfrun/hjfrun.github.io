---
layout:     post
title:      表情输入面板及输入展示
subtitle:   
date:       2017-03-29 23:39:35
author:     "hjfrun"

header-img: ""
catalog: false
tags:
    - iOS
    - Foundation
---



用户输入表情面板，像QQ以及微信等其他的社交软件一样，提供一个表情输入面板，把用户点击的表情显示到一个可以自动增高的UITextView上，然后点击发送，再把这个表情显示在UILabel或者其他什么地方。

今天只是抽空搭了个框架。完成到这一步。显示效果如下图。后续还会继续完善。

![](../img/in-post/emotion-input-view-1.png)





今天需要梳理到的地方是。把表情对应的图片插入到文本中。方法是把表情-图片变成属性文字的附件，然后作为属性文字插入。一个简单的实现如下：

```objc
- (void)insertTextView:(UITextView *)textView withEmotion:(Emotion *)emotion
{
    NSTextAttachment *attach = [[NSTextAttachment alloc] init];
    attach.image = [UIImage imageNamed:emotion.png];
    attach.bounds = CGRectMake(0, -5, 20, 20);
    NSAttributedString *attriStr = [NSAttributedString attributedStringWithAttachment:attach];

    NSMutableAttributedString *attriStrM = [[NSMutableAttributedString alloc] initWithAttributedString:textView.attributedText];
    
    NSUInteger loc = textView.selectedRange.location;
    
    [attriStrM insertAttributedString:attriStr atIndex:loc];
    
    textView.attributedText = attriStrM;
    textView.selectedRange = NSMakeRange(loc + 1, 0);
}
```



[Github地址](https://github.com/hjfrun/ImageTextLayoutDemo)