---
layout:     post
title:      文本和自定义表情的相互转换
subtitle:   
date:       2017-04-02 21:27:35
author:     hjfrun

header-img: ""
catalog: false
tags:
    - iOS
    - Foundation
---



之前做评论，用户点击表情面板，输入表情。然后发送给服务端，这个时候并不是真是把属性文字全部发给服务端。而是要转换为纯文本发送，把图片表情替换为`[哈哈]`这种样式。加载评论的时候，需要把接收的评论里面的文本表情替换为图片，也就是用属性文字来表示。我们项目里面之前就有这样的功能，但是没有正则表达式，是手写的搜索判断。感觉很恶心，今天用正则表达式又重新实现了一遍。效果如下图：

![](/img/in-post/2017-04-02 21_06_50.gif)

**主要思路**

* 表情转文本：遍历属性文字，会得到一个个表情的range，然后拿到属性文字，解析字典里面的`NSAttachment`字段，将其转换为对应的文本。
* 文本转表情：遍历纯文本，使用正则表达式匹配哪些是表情。使用属性文字挨个拼接到一起。



主要实现代码如下：

需要注意的是，我这里的`EmotionAttachment`类继承自系统的`NSTextAttachment`类并绑定了一个表情模型。

```objc
/**
 在这里把带表情的属性文字转换为文本

 @param attriString 带表情的属性文字
 @return 纯文本
 */
- (NSString *)textFromAttributedText:(NSAttributedString *)attriString
{
    NSMutableString *resultString = [NSMutableString string];
    [attriString enumerateAttributesInRange:NSMakeRange(0, attriString.length) options:NSAttributedStringEnumerationLongestEffectiveRangeNotRequired usingBlock:^(NSDictionary<NSString *,id> * _Nonnull attrs, NSRange range, BOOL * _Nonnull stop) {
        id obj = attrs[@"NSAttachment"];
        if (!obj) {
          	// 不是表情
            [resultString appendString:[attriString attributedSubstringFromRange:range].string];
        } else {
          	// 是表情
            EmotionAttachment *emotionAttach = (EmotionAttachment *)obj;
            [resultString appendString:emotionAttach.emotion.chs];
        }
    }];
    
    return resultString;
}


/**
 文本转换为带表情的属性文字

 @param text 纯文本
 */
- (NSAttributedString *)attributedTextFromText:(NSString *)text
{
    NSMutableAttributedString *attriStringM = [[NSMutableAttributedString alloc] init];
    
    NSString *emotionPattern = @"\\[\\w+\\]";	// 正则表达式的模式
  	// 生成正则表达式
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:emotionPattern options:NSRegularExpressionCaseInsensitive error:nil];	
    NSArray *resultsArray = [regex matchesInString:text options:kNilOptions range:NSMakeRange(0, text.length)];
    
    NSUInteger loc = 0;	// 记录每次开始的位子，挨个拼接
    for (NSTextCheckingResult *result in resultsArray) {
        // NSLog(@"range : %@", NSStringFromRange(result.range));
        
        NSUInteger length = result.range.location - loc;
        
        [attriStringM appendAttributedString:[[NSAttributedString alloc] initWithString:[text substringWithRange:NSMakeRange(loc, length)]]];
        
        NSString *chs = [text substringWithRange:result.range];
        
        [attriStringM appendAttributedString:[self emotionAttachWithChs:chs]];
        
        loc = result.range.location + result.range.length;
    }
  	// 注意尾部的文本也要拼接到后面，不要遗漏
    [attriStringM appendAttributedString:[[NSAttributedString alloc] initWithString:[text substringFromIndex:loc]]];
     
    return attriStringM;
}


/**
 根据匹配的[]的内容去返回表情属性文字

 @param chs  表情对应的描述字符
 @return 表情属性字符串
 */
- (NSAttributedString *)emotionAttachWithChs:(NSString *)chs
{
    Emotion *emotion = [EmotionManager emotionFromChs:chs];
    if (!emotion) {
        return [[NSAttributedString alloc] initWithString:chs];
    }
    EmotionAttachment *emotionAttach = [[EmotionAttachment alloc] init];
    emotionAttach.emotion = emotion;
    emotionAttach.bounds = CGRectMake(0, -5, self.label.font.lineHeight, self.label.font.lineHeight);
    return [NSAttributedString attributedStringWithAttachment:emotionAttach];
}
```



说明：

1. 我这里用的是挨个拼接的方法，在只是单独处理表情的情况有效，比较便捷，如果不是文本中还需要处理URL、@#特殊标记的字段，则使用起来不是很凑效。需要想别的办法；
2. 给属性文字绑定了模型才能够通过解析出来的属性文字拿到模型，然后拼接对应的文本。使用继承或者分类都可以做到。我这里使用的是继承；
3. 这里处理的表情模型还是简单的，使用[]模式去匹配表情找表情图片不一定准确。特别是聊天的场景，一个表情文本经常能对应很多的表情。需要更精细的处理。

[Github地址](https://github.com/hjfrun/ImageTextLayoutDemo)
