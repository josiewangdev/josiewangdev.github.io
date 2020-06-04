---
layout: post
title: UIScrollView多层嵌套中的联动效果实现
tags: Project
categories: Tech
cover: images/multi_scrollview/view_hierarchy.jpg
---

* 在项目中，经常会出现多层滚动视图嵌套的才能实现的UI需求。但是多层滚动视图的嵌套，通常会引发手势不连贯，用户操作滑动不顺畅等问题。因为通常情况下，只有最上层滚动视图能够响应手势动作。如果想让ScrollView的滑动效果能够平滑的由上层穿透到下层，无缝滚动连接，则需要将下层视图的手势穿透响应打开。

例如我现在所做的项目中，有如下UI设计需求，只能由多层的ScrollView嵌套去实现。(在这里，我把TableView与CollectionView都算作ScrollView的一种)

<img src="/images/multi_scrollview/multi_scrollview.gif"/>

<img src="/images/multi_scrollview/view_hierarchy.jpg"/>

<br>
### 代码实现 ###
<br>

* 先实现UIGestureRecognizerDelegate协议的方法，UIScrollView及其子类已经实现了该方法，只需将返回值返回为YES/true

{% highlight objc %}
//ObjC
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer {
    return YES;
}
{% endhighlight %}

{% highlight swift %}
//Swift3
func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
    return true
}
{% endhighlight %}

{% highlight objc %}
//Objective-C
- (BOOL)gestureRecognizer:(UIPanGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UISwipeGestureRecognizer *)otherGestureRecognizer 
{
    return YES;
}
{% endhighlight %}

* 其次，我们需要做的，仅仅是判断何时不让某个scrollview改变偏移量即可。

{% highlight swift %}
//该项目是用Swift3实现的，在这里只贴一下Swift的实现代码

fileprivate enum ScrollViewOffsetType {
    case max
    case min
    case center
}

//MainScrollView的UIScrollViewDelegate协议方法

func scrollViewDidScroll(_ scrollView: UIScrollView) {

    if scrollView.contentOffset.y + scrollView.bounds.height >= scrollView.contentSize.height {
        mainScrollViewOffsetType = .max
    }else if scrollView.contentOffset.y <= 0 {
        mainScrollViewOffsetType = .min
    }else {
        mainScrollViewOffsetType = .center
    }

    //SlideTabView即为上图所示的SubScrollView，是我写的一个嵌套子滑动视图的一个通用控件
    if slideTabView.currentIndex == 0 && productCollectionViewOffsetType == .center {
        scrollView.contentOffset = CGPoint(x: scrollView.contentOffset.x, y: scrollView.contentSize.height - scrollView.bounds.height)
    }

    if slideTabView.currentIndex == 1 && buyerCommentTableViewOffsetType == .center {
        scrollView.contentOffset = CGPoint(x: scrollView.contentOffset.x, y: scrollView.contentSize.height - scrollView.bounds.height)
    }
}

//以SubScrollView中的子滚动视图CollectionView为例，当其滚动到顶部，开启MainScrollView的滚动，在这里ProductCollectionView也是我写的一个通用组件，并且我将它的UIScrollViewDelegate代理协议方法通过block函数回调实现

weak var weakSelf = self
productCollectionView.handleCollectionViewDidScroll = { (scrollView) in

    if scrollView.contentOffset.y <= 0 {
        weakSelf?.productCollectionViewOffsetType = .min
    } else {
        weakSelf?.productCollectionViewOffsetType = .center
    }

    if weakSelf?.mainScrollViewOffsetType == .min {
        scrollView.contentOffset = CGPoint.zero
    }

    if weakSelf?.mainScrollViewOffsetType == .center {
        scrollView.contentOffset = CGPoint.zero
    }

}

{% endhighlight %}

通过设置子滚动视图与主滚动视图的代理协议方法，确定了他们之间滚动时偏移量的关系后，平滑的多ScrollView滚动效果就实现了。

[[参考博文] iOS解决方案：多个scrollview联动](http://www.jianshu.com/p/42479a0e8ac6)
