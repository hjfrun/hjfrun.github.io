---
layout:     post
title:      "说点什么"
subtitle:   ""
date:       2017-03-27 23:36:35
author:     "hjfrun"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 生活
---



第二篇内容，占位看看效果，明天就移除掉。



以下测试内容。。。



```objective-c
- (void)setIntroduceContentWithString:(NSString *)str
{
    CGFloat marginDistance = 12.f;
    CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
    CGFloat textShowWidth = screenWidth - (marginDistance * 2);
    
    CGFloat textRowHeight = [self calculateTextLabelHeightWithString:[self lineSpaceAttributeStringWithStr:@"用于标尺"] andWidth:textShowWidth];
    
    NSString *moreText = @"...全文";
    //所占宽度
    CGFloat moreTextWidth = [self calculateTextLabelWidthWithString:moreText
                                                          andHeight:textRowHeight];
    
    NSAttributedString *introText = [self lineSpaceAttributeStringWithStr:str];
    CGFloat attrTextHeight = [self calculateTextLabelHeightWithString:introText andWidth:textShowWidth];
    
    CGSize textSize = CGSizeMake(textShowWidth, textRowHeight);
    NSArray *truncatedArray = [self truncatedArray:str forFont:self.contentLabel.font size:textSize];
    
    if (attrTextHeight <= textRowHeight * 3) {
        //小于，等于 三行，直接展示即可
        self.contentLabelHeightConstraint.constant = attrTextHeight;
        self.contentLabel.attributedText = introText;
        
        //设置unfold按钮不可点击
        self.unfoldButton.hidden = YES;
    } else {
        //大于三行
        self.unfoldButton.hidden = NO;
        
        if (!self.isUnfold) {//收起
            //先拼接前两行
            NSMutableString *introductText = [NSMutableString stringWithFormat:@""];
            for (NSUInteger index = 0; index < 2; index++) {
                if ([truncatedArray objectAtIndexIfIndexInBounds:index]) {
                    [introductText appendString:[truncatedArray objectAtIndexIfIndexInBounds:index]];
                }
            }
            
            CGFloat maxIntroWidth = textShowWidth - moreTextWidth - 1;
            truncatedArray = [self truncatedArray:[truncatedArray objectAtIndexIfIndexInBounds:2]
                                          forFont:self.contentLabel.font
                                             size:CGSizeMake(maxIntroWidth, textRowHeight)];
            
            if (truncatedArray.count > 0) {
                [introductText appendString:[truncatedArray objectAtIndexIfIndexInBounds:0]];
                
            }
            
            if ([[introductText substringFromIndex:introductText.length - 1] isEqualToString:@"\n"]) {
                [introductText insertString:moreText atIndex:introductText.length - 1];
            } else {
                [introductText appendString:moreText];
            }
            
            NSInteger loc = [introductText rangeOfString:moreText options:NSCaseInsensitiveSearch range:NSMakeRange(0, introductText.length)].location;
            
            CGFloat textHeight = [self calculateTextLabelHeightWithString:[self lineSpaceAttributeStringWithStr:introductText] andWidth:textShowWidth];
            self.contentLabelHeightConstraint.constant = textHeight;
            
            NSMutableAttributedString *mutableAttString = [self lineSpaceAttributeStringWithStr:introductText];
            [mutableAttString addAttribute:NSForegroundColorAttributeName value:[UIColor colorWithRGBString:@"00AEEF"] range:NSMakeRange(loc + 3, 2)];
            
            self.contentLabel.attributedText = mutableAttString;
        } else {//展开
            CGFloat textHeight = [self calculateTextLabelHeightWithString:introText andWidth:textShowWidth];
            self.contentLabelHeightConstraint.constant = textHeight;
            self.contentLabel.attributedText = introText;
        }
        
    }
}

```




