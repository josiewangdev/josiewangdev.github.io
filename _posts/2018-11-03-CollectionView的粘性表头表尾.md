---
layout: post
title: Collection view 的粘性表头表尾 · Sticky header/footer in collection view
tags: iOS
categories: Dev
---

The **sticky header and footer** was added for `UICollectionView` in iOS 9.0, although it’s not enabled by default and only available through following steps.

1\. Make your collection view’s `collectionViewLayout` property is an instance of `UICollectionViewFlowLayout`.

2\. Set the flow layout’s `sectionHeadersPinToVisibleBounds` property to `true`. 

3\. Similar property to make footers behave the same way: `sectionFootersPinToVisibleBounds`.

在 iOS 9.0 中，为 `UICollectionView` 添加了**粘性表头表尾**（类似于table view header）的效果，但是这并不是默认设置，实现它需要以下几步。

1\. 你设置集合视图布局的 `collectionViewLayout` 属性是继承于 `UICollectionViewFlowLayout` 的.

2\. 设置布局的 `sectionHeadersPinToVisibleBounds` 属性为 `true`.  

3\. 同样的可以使表尾呈现粘性效果的属性为: `sectionFootersPinToVisibleBounds`。 

---

{% highlight swift %}
if let layout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    layout.sectionHeadersPinToVisibleBounds = true
    layout.sectionFootersPinToVisibleBounds = true
}
{% endhighlight %}

[[Ref.参考] Apple Doc](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1617699-sectionheaderspintovisiblebounds)