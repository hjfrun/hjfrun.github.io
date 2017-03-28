---
layout:     post
title:      可以折叠展开的UILabel
subtitle:   
date:       2017-03-28 22:52:35
author:     "hjfrun"
header-img: ""
catalog: false
tags:
    - iOS
---



之前有做到一个feature，我们的电影有个模块，叫剧情简介。要展示一长段文本。默认折叠状态最多显示3行，3行放不下的时候，在末尾添加【…全文】用户在点击了这长段文本的时候把这个label进行展开。跟网友提的这个问题有点相似 https://segmentfault.com/q/1010000004452279。

效果如下图：

![collapse label](https://github.com/hjfrun/hjfrun.github.io/blob/master/img/collapse%20label.gif)



折腾了半天实现的方法还挺复杂的，不是很好做。现在把以前做的梳理下。把主干方法抽出来，并没有做完整的情况判断，也缺乏安全监测。不建议直接拿去使用。可以根据需求定制。使用到CoreText框架。



```objective-c
// 使用以下方法把string拆分为一行行文本数组，size参数为给的一行文本框大小
- (NSArray *)splitWithString:(NSString *)string size:(CGSize)size
{
    if (!string || string.length == 0) {
        return nil;
    }
    
    NSMutableAttributedString *attriStringM = [[NSMutableAttributedString alloc] initWithString:string attributes:self.attrs];
    
    NSMutableArray *textLines = @[].mutableCopy;
    
    NSString *textLine = [NSString string];
    
    CTFramesetterRef frameSetter;
    CFRange fitRange;
    // 不断的对string截取一行行文本，结果存放到数组
    while (attriStringM.length) {
        frameSetter = CTFramesetterCreateWithAttributedString((__bridge CFAttributedStringRef)attriStringM);
        
        CTFramesetterSuggestFrameSizeWithConstraints(frameSetter, CFRangeMake(0, 0), NULL, size, &fitRange);
        CFRelease(frameSetter);
        
        textLine = [[attriStringM attributedSubstringFromRange:NSMakeRange(0, fitRange.length)] string];
        
        [textLines addObject:textLine];
        
        [attriStringM setAttributedString:[attriStringM attributedSubstringFromRange:NSMakeRange(fitRange.length, attriStringM.string.length - fitRange.length)]];
    }
    
    return textLines;
}

```



主要思想：

让label高度自适应，通过设置内容使得label自己调节自己的高度。当然了，也可以使用别的方法布局。如果还要跟其他控件进行组合的话，则情况可能更加复杂。



根据给定的一长段文本，生成两个属性文字。一个是```折叠形式```，一个是```展开形式```。

```折叠形式```：把文本按照上面的方法，拆成一行行。其实还可以做简单点，至需要知道前面3行即可。把两面两行的内容加起来。然后计算第三行的宽度减去【…全文】的宽度，剩下的可以得出第三行可以放置多少文字。同样也可以使用上面的方法进行计算。然后把第三行的文本以属性文字的形式append上去。在把【…全文】append上去。

```展开形式```：属性文字按直接展示。



一个粗糙的实现可以参考我的[GitHub](https://github.com/hjfrun/UILabelExpandAll)。一定注意不能拿来直接使用，一定要进行各种情况的判断。等有时间了，再来对这个项目进行详细的扩充和说明。









