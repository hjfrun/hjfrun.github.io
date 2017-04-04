---
layout:     post
title:      表情键盘布局
subtitle:   分页、删除按钮
date:       2017-04-04 23:07:35
author:     hjfrun
header-img: 
catalog: false
tags:
    - iOS
    - 表情键盘
---



有表情键盘的地方，经常需要把表情分页，并在最后一个按钮地方设置为删除。就像微信、微博里面的表情键盘一样。不管页面是怎么样的，始终把最后一个【delete】放在最后一个位置。之前改造过我们项目里面的表情键盘。其实就是写一个`collectionView`的`layout`。我的实现大致是下面这个样子的。如果需要更精细的控制可以继续完善。



**EmotionLayout.h**

```objc
@interface EmotionLayout : UICollectionViewFlowLayout

@property (nonatomic, assign, readonly) NSUInteger maxCols;
@property (nonatomic, assign, readonly) NSUInteger maxRows;

- (instancetype)initWithMaxCols:(NSUInteger)maxCols maxRows:(NSUInteger)maxRows;
```

**EmotionLayout.m**

```objc
@interface EmotionLayout()

@property (nonatomic, strong) NSMutableArray *attriArray;
@property (nonatomic, assign) CGFloat itemWidth;

@end

@implementation EmotionLayout

- (NSMutableArray *)attriArray
{
    if (_attriArray == nil) {
        _attriArray = @[].mutableCopy;
    }
    return _attriArray;
}

- (instancetype)init
{
    return [self initWithMaxCols:7 maxRows:3];
}

- (instancetype)initWithMaxCols:(NSUInteger)maxCols maxRows:(NSUInteger)maxRows
{
    if (self = [super init]) {
        _maxCols = maxCols;
        _maxRows = maxRows;
    }
    return self;
}

- (void)prepareLayout
{
    [super prepareLayout];
    
    self.itemWidth = [UIScreen mainScreen].bounds.size.width / self.maxCols;
    
    self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
    self.itemSize = CGSizeMake(self.itemWidth, self.itemWidth);
    self.minimumLineSpacing = 0;
    self.minimumInteritemSpacing = 0;
    
    [self.attriArray removeAllObjects];
    
    NSInteger count = [self.collectionView numberOfItemsInSection:0];
    
    for (NSInteger i = 0; i < count; i++) {
        NSIndexPath *indexPath = [NSIndexPath indexPathForItem:i inSection:0];
        UICollectionViewLayoutAttributes *attrs = [self layoutAttributesForItemAtIndexPath:indexPath];
        [self.attriArray addObject:attrs];
    }
}

- (CGSize)collectionViewContentSize
{
    NSInteger emotionCount = [self.collectionView numberOfItemsInSection:0];
    
    CGFloat contentWidth = self.collectionView.bounds.size.width * ((emotionCount - 1) / self.maxCols / self.maxRows + 1);
    CGFloat contentHeight = self.collectionView.bounds.size.height;
    
    return CGSizeMake(contentWidth, contentHeight);
}

- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect
{
    return self.attriArray;
}

- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath
{
    UICollectionViewLayoutAttributes *attributes = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    
    NSInteger item = indexPath.item;
    
    if (item == [self.collectionView numberOfItemsInSection:0] - 1) {
        item = (item / self.maxRows / self.maxCols + 1) * self.maxCols * self.maxRows - 1;
    }
    
    NSInteger page = item / self.maxCols / self.maxRows;
    
    CGFloat attributesX = (item % self.maxCols) * self.itemWidth + page * self.itemWidth * self.maxCols;
    CGFloat attributesY = (item / self.maxCols) * self.itemWidth - page * self.itemWidth * self.maxRows;
    
    attributes.frame = CGRectMake(attributesX, attributesY, self.itemWidth, self.itemWidth);
    
    return attributes;
}

@end
```

配置数据源，每页插入删除按钮：

```objc
// 给在表情里面插入删除按钮
- (NSArray *)emotions
{
    if (_emotions == nil) {
        NSMutableArray *arrayM = @[].mutableCopy;
        NSArray *pureEmotions = [EmotionManager emotionList];
        Emotion *deleteEmotion = [Emotion new];
        deleteEmotion.png = @"emotion_delete";
        deleteEmotion.chs = @"";
        
        NSInteger emotionPageSize = self.maxRows * self.maxCols - 1;
        NSUInteger pageCount = pureEmotions.count / emotionPageSize;
        
        for (NSInteger i = 0; i < pageCount; i++) {
            [arrayM addObjectsFromArray:[pureEmotions subarrayWithRange:NSMakeRange(i * emotionPageSize, emotionPageSize)]];
            [arrayM addObject:deleteEmotion];
        }
        
        if (pureEmotions.count % emotionPageSize) {
            [arrayM addObjectsFromArray:[pureEmotions subarrayWithRange:NSMakeRange(pageCount * emotionPageSize, pureEmotions.count % emotionPageSize)]];
            [arrayM addObject:deleteEmotion];
        }
        _emotions = arrayM;
    }
    
    return _emotions;
}
```

键盘的`deleteClickedBlock`

```objc
[self.inputView setDeleteClickedBlock:^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    [strongSelf.textView deleteBackward];
}];
```

效果如下图：



![](../img/in-post/emotion_layout_1.png)



参考：

[Github地址](https://github.com/hjfrun/ImageTextLayoutDemo)