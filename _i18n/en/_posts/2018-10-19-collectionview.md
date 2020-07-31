---
layout: post
title: Collection View
tags: iOS
categories: Dev
---

## Collection View

---

#### 1. [自定义UICollectionView Layout](#)
#### 2. [Sticky header/footer in collection view](#)

---

<br>

> #### 1. 自定义UICollectionView Layout {#}

自定义UICollectionViewLayout子类CustomCollectionViewLayout

1). 重写prepareLayout，存储布局信息。

2). 基于prepareLayout方法中的布局信息，使用collectionViewContentSize方法返回UICollectionView的内容尺寸。

3). 使用layoutAttributesForElementsInRect返回指定区域cell、Supplementary View和Decoration View的布局属性。

4). 获取对应indexPath的UICollectionViewLayoutAttributes，这个类其实专门用来存储视图的内容，例如frame、size、apha、hidden等等，layout最后会拿着这些frame设置给对应的视图。

5). 存储attributes到二维数组layoutInfoArr当中。

6). 计算出内容尺寸并保存到全局变量contentSize当中。

```objc
- (void)prepareLayout{
    [super prepareLayout];
    NSMutableArray *layoutInfoArr = [NSMutableArray array];
    NSInteger maxNumberOfItems = 0;
    //获取UICollectionViewLayoutAttributes布局信息
    NSInteger numberOfSections = [self.collectionView numberOfSections];
    for (NSInteger section = 0; section < numberOfSections; section++){
        NSInteger numberOfItems = [self.collectionView numberOfItemsInSection:section];
        NSMutableArray *subArr = [NSMutableArray arrayWithCapacity:numberOfItems];
        for (NSInteger item = 0; item < numberOfItems; item++){
            NSIndexPath *indexPath = [NSIndexPath indexPathForItem:item inSection:section];
            UICollectionViewLayoutAttributes *attributes = [self layoutAttributesForItemAtIndexPath:indexPath];
            [subArr addObject:attributes];
        }
        if(maxNumberOfItems < numberOfItems){
            maxNumberOfItems = numberOfItems;
        }
        //添加到二维数组
        [layoutInfoArr addObject:[subArr copy]];
    }
    //存储布局信息
    self.layoutInfoArr = [layoutInfoArr copy];
    //保存内容尺寸
    self.contentSize = CGSizeMake(maxNumberOfItems*(self.itemSize.width+self.interitemSpacing)+self.interitemSpacing, numberOfSections*(self.itemSize.height+self.lineSpacing)+self.lineSpacing);
}
```

而上面代码中，获取UICollectionViewLayoutAttributes是通过layoutAttributesForItemAtIndexPath方法

```swift
- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewLayoutAttributes *attributes = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    //每一组cell为一行
    attributes.frame = CGRectMake((self.itemSize.width+self.interitemSpacing)*indexPath.row+self.interitemSpacing, (self.itemSize.height+self.lineSpacing)*indexPath.section+self.lineSpacing, self.itemSize.width, self.itemSize.height);
    return attributes;
}
```

覆写collectionViewContentSize。

collectionViewContentSize返回内容尺寸给UICollectionView。注意这个方法返回的尺寸是给UICollectionView这个继承于UIScrollView的视图作为contentSize，不是UICollectionView的视图尺寸。正是因为这一点，我们自定义layout如果想让它只能横向滑动，只需要将这个size.height设置成collectionView.height就行了。
这个方法会多次调用，所以最好是在prepareLayout里就计算好。
在CustomCollectionViewLayout类中，我们只需要返回前面计算好的内容尺寸就行了。

```swift
- (CGSize)collectionViewContentSize{
    return self.contentSize;
}
```

覆写layoutAttributesForElementsInRect方法

此方法需要返回一组UICollectionViewLayoutAttributes类型对象。它代表着在这个指定的区域中，我们需要显示cell、Supplementary View和Decoration View中哪些视图，而这些视图的属性则保存UICollectionViewLayoutAttributes中。
此方法会多次调用，为了更好的性能，在这个方法当中，我们使用的UICollectionViewLayoutAttributes最好是在prepareLayout已经布局好的信息。

在CustomCollectionViewLayout，我们遍历二维数组，找出了与指定区域有交接的UICollectionViewLayoutAttributes对象放到一个数组中，然后返回。

```swift
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect{
    NSMutableArray *layoutAttributesArr = [NSMutableArray array];
    [self.layoutInfoArr enumerateObjectsUsingBlock:^(NSArray *array, NSUInteger i, BOOL * _Nonnull stop) {
        [array enumerateObjectsUsingBlock:^(UICollectionViewLayoutAttributes *obj, NSUInteger idx, BOOL * _Nonnull stop) {
            if(CGRectIntersectsRect(obj.frame, rect)) {
                [layoutAttributesArr addObject:obj];
            }
        }];
    }];
    return layoutAttributesArr;
}
```

I modified his code to insure that items are ALWAYS left aligned. I found that if an item ended up on a line by itself, it would be centered by the flow layout. I made the following changes to address this issue.

This situation would only ever occur if you have cells that vary in width, which could result in a layout like the following. The last line always left aligns due to the behavior of UICollectionViewFlowLayout, the issue lies in items that are by themselves in any line but the last one.


```swift
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect {
    NSArray* attributesToReturn = [super layoutAttributesForElementsInRect:rect];
    for (UICollectionViewLayoutAttributes* attributes in attributesToReturn) {
        if (nil == attributes.representedElementKind) {
            NSIndexPath* indexPath = attributes.indexPath;
            attributes.frame = [self layoutAttributesForItemAtIndexPath:indexPath].frame;
        }
    }
    return attributesToReturn;
}

- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath {
    UICollectionViewLayoutAttributes* currentItemAttributes =
    [super layoutAttributesForItemAtIndexPath:indexPath];

    UIEdgeInsets sectionInset = [(UICollectionViewFlowLayout *)self.collectionView.collectionViewLayout sectionInset];

    if (indexPath.item == 0) { // first item of section
        CGRect frame = currentItemAttributes.frame;
        frame.origin.x = sectionInset.left; // first item of the section should always be left aligned
        currentItemAttributes.frame = frame;

        return currentItemAttributes;
    }

    NSIndexPath* previousIndexPath = [NSIndexPath indexPathForItem:indexPath.item-1 inSection:indexPath.section];
    CGRect previousFrame = [self layoutAttributesForItemAtIndexPath:previousIndexPath].frame;
    CGFloat previousFrameRightPoint = previousFrame.origin.x + previousFrame.size.width + kMaxCellSpacing;

    CGRect currentFrame = currentItemAttributes.frame;
    CGRect strecthedCurrentFrame = CGRectMake(0,
                                              currentFrame.origin.y,
                                              self.collectionView.frame.size.width,
                                              currentFrame.size.height);

    if (!CGRectIntersectsRect(previousFrame, strecthedCurrentFrame)) { // if current item is the first item on the line
        // the approach here is to take the current frame, left align it to the edge of the view
        // then stretch it the width of the collection view, if it intersects with the previous frame then that means it
        // is on the same line, otherwise it is on it's own new line
        CGRect frame = currentItemAttributes.frame;
        frame.origin.x = sectionInset.left; // first item on the line should always be left aligned
        currentItemAttributes.frame = frame;
        return currentItemAttributes;
    }

    CGRect frame = currentItemAttributes.frame;
    frame.origin.x = previousFrameRightPoint;
    currentItemAttributes.frame = frame;
    return currentItemAttributes;
}
```

The other solutions in here not work properly when the line is composed by only 1 item or are over complicated.

Based on the example given by Ryan, I changed the code to detect a new line by inspecting the Y position of the new element. Very simple and quick in performance.

```swift
class LeftAlignedCollectionViewFlowLayout: UICollectionViewFlowLayout {

    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        let attributes = super.layoutAttributesForElements(in: rect)

        var leftMargin = sectionInset.left
        var maxY: CGFloat = -1.0
        attributes?.forEach { layoutAttribute in
            if layoutAttribute.frame.origin.y >= maxY {
                leftMargin = sectionInset.left
            }

            layoutAttribute.frame.origin.x = leftMargin

            leftMargin += layoutAttribute.frame.width + minimumInteritemSpacing
            maxY = max(layoutAttribute.frame.maxY , maxY)
        }

        return attributes
    }
}
```

If you want to have supplementary views keep their size, add the following at the top of the closure in the forEach call:

```swift
guard layoutAttribute.representedElementCategory == .cell else {
    return
}
```

---

<br>

> #### 2. Sticky header/footer in collection view {#}

The **sticky header and footer** was added for `UICollectionView` in iOS 9.0, although it’s not enabled by default and only available through following steps.

1). Make your collection view’s `collectionViewLayout` property is an instance of `UICollectionViewFlowLayout`.

2). Set the flow layout’s `sectionHeadersPinToVisibleBounds` property to `true`. 

3). Similar property to make footers behave the same way: `sectionFootersPinToVisibleBounds`.

```swift
if let layout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    layout.sectionHeadersPinToVisibleBounds = true
    layout.sectionFootersPinToVisibleBounds = true
}
```

---

<br>

#### 参考链接

* [StackOverflow - How do you determine spacing between cells in UICollectionView flowLayout](https://stackoverflow.com/questions/13017257/how-do-you-determine-spacing-between-cells-in-uicollectionview-flowlayout)
* [Github - UICollectionViewLeftAlignedLayout-Swift](https://github.com/fanpyi/UICollectionViewLeftAlignedLayout-Swift)
* [UICollectionView之自定义Layout](http://www.liuchungui.com/blog/2015/11/06/uicollectionviewzhi-zi-ding-yi-layout/)
* [StackOverflow - Left Align Cells in UICollectionView](https://stackoverflow.com/questions/22539979/left-align-cells-in-uicollectionview)
* [Apple Doc](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1617699-sectionheaderspintovisiblebounds)


