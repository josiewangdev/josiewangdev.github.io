---
layout: post
title: 集合视图
tags: iOS
categories: Dev
---

## 集合视图

---

#### 1. [集合视图的粘性表头表尾](#p1)
#### 2. [自定义集合视图的布局](#p2)

---

<br>

> #### 1. 集合视图的粘性表头表尾 {#p1}

iOS 9.0 之后，为 `UICollectionView` 添加了**粘性表头表尾**（类似于table view header）的效果，但是这并不是默认设置，实现粘性效果需要以下几步

**1).** 设置集合视图布局的 `collectionViewLayout` 属性继承于 `UICollectionViewFlowLayout`

**2).** 实现表头粘性效果, 需要设置布局属性 `sectionHeadersPinToVisibleBounds` 的值为 `true`

**3).** 实现表尾呈现粘性效果, 则设置属性 `sectionFootersPinToVisibleBounds` 的值为 `true`

```swift
if let layout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    layout.sectionHeadersPinToVisibleBounds = true
    layout.sectionFootersPinToVisibleBounds = true
}
```

---

<br>

> #### 2. 自定义集合视图的布局 {#p2}

众所周知系统自定义的 `UICollectionViewFlowLayout` 布局的排列方式是居中对齐的。 如果有这样一个开发需求 -- 实现一个左对齐排列的集合视图时, 系统定义的视图布局已经不能满足我们的开发需求了, 我们就需要自定义一个集合视图的布局。 

自定义一个 Collection View Layout 需要以下几步: 

**1).** 创建 `LeftAlignedCollectionViewFlowLayout` 类继承于 `UICollectionViewFlowLayout`。

**2).** 获取对应 **indexPath** 的 **UICollectionViewLayoutAttributes** ，修改或自定义这些attributes到二维数组中。`UICollectionViewLayoutAttributes` 这个类其实专门用来存储视图的内容，例如 frame、size、apha、
hidden 等等属性，存储的视图属性除了 **Cell** 的以外, 也包括 **Supplementary View** 和 **Decoration View**。

**3).** 使用 `layoutAttributesForElementsInRect` 方法返回自定义的attributes二维数组。 layout最后会使用attributes中设置的frame等值赋给对应的视图。

```swift
class LeftAlignedCollectionViewFlowLayout: UICollectionViewFlowLayout {

    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        //  获取attributes
        let attributes = super.layoutAttributesForElements(in: rect)
        var leftMargin = sectionInset.left
        var maxY: CGFloat = -1.0

        attributes?.forEach { layoutAttribute in

            guard layoutAttribute.representedElementCategory == .cell else {
                //  如果想要修改 supplementary views 等其他视图的属性, 可以在这里实现
                return
            }

            if layoutAttribute.frame.origin.y >= maxY {
                leftMargin = sectionInset.left
            }

            //  修改 cell 视图 attributes的值
            layoutAttribute.frame.origin.x = leftMargin
            leftMargin += layoutAttribute.frame.width + minimumInteritemSpacing
            maxY = max(layoutAttribute.frame.maxY , maxY)
            
        }

        //  返回自定义 attributes 数组
        return attributes

    }
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



