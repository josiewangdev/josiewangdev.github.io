---
layout: post
title: "UIScrollViewWillEndDraggingWithVelocity的用法"
tags: Project
categories: [Tech]
---

**实例 Table View 中图片加载逻辑的优化**

<br>

当用户手动 drag table view 的时候，会加载 cell 中的图片；
在用户快速滑动的减速过程中，不加载过程中 cell 中的图片（但文字信息还是会被加载，只是减少减速过程中的网络开销和图片加载的开销）；
在减速结束后，加载所有可见 cell 的图片（如果需要的话）；
问题 1：

前面提到，刚开始拖动的时候，dragging 为 YES，decelerating 为 NO；decelerate 过程中，dragging 和 decelerating 都为 YES；decelerate 未结束时开始下一次拖动，dragging 和 decelerating 依然都为 YES。所以无法简单通过 table view 的 dragging 和 decelerating 判断是在用户拖动还是减速过程。

解决这个问题很简单，添加一个变量如 userDragging，在 willBeginDragging 中设为 YES，didEndDragging 中设为 NO。那么 tableView: cellForRowAtIndexPath: 方法中，是否 load 图片的逻辑就是：

<br>

{% highlight swift %}
if (!self.userDragging && tableView.decelerating) {  
    cell.imageView.image = nil;
} else {
    // code for loading image from network or disk
}
{% endhighlight %}

问题 2：

这么做的话，decelerate 结束后，屏幕上的 cell 都是不带图片的，解决这个问题也不难，你需要一个形如 loadImageForVisibleCells 的方法，加载可见 cell 的图片：

<br>

{% highlight swift%}
- (void)loadImageForVisibleCells
{
    NSArray *cells = [self.tableView visibleCells];
    for (GLImageCell *cell in cells) {
        NSIndexPath *indexPath = [self.tableView indexPathForCell:cell];
        [self setupCell:cell withIndexPath:indexPath];
    }
}
{% endhighlight %}

<br>

问题 3：

这个问题可能不容易被发现，在减速过程中如果用户开始新的拖动，当前屏幕的 cell 并不会被加载（前文提到的调用顺序问题导致），而且问题 1 的方案并不能解决问题 3，因为这些 cell 已经在屏上，不会再次经过 cellForRowAtIndexPath 方法。虽然不容易发现，但解决很简单，只需要在 scrollViewWillBeginDragging: 方法里也调用一次 loadImageForVisibleCells 即可。

**再优化**

如果你的 App 只有图片没有文字。配合 SDWebImage，可以再优化了一下这个方法以提升用户体验：

* 如果内存中有图片的缓存，减速过程中也会加载该图片
* 如果图片属于 targetContentOffset 能看到的 cell，正常加载，这样一来，快速滚动的最后一屏出来的的过程中，用户就能看到目标区域的图片逐渐加载
* 你可以尝试用类似 fade in 或者 flip 的效果缓解生硬的突然出现（尤其是像只有图片的 App）

{% highlight swift %}
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    self.targetRect = nil;
    [self loadImageForVisibleCells];
}

- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset
{
    CGRect targetRect = CGRectMake(targetContentOffset->x, targetContentOffset->y, scrollView.frame.size.width, scrollView.frame.size.height);
    self.targetRect = [NSValue valueWithCGRect:targetRect];
}

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    self.targetRect = nil;
    [self loadImageForVisibleCells];
}
{% endhighlight %}

<br>

是否需要加载图片的逻辑：

{% highlight swift %}
BOOL shouldLoadImage = YES;  
if (self.targetRect && !CGRectIntersectsRect([self.targetRect CGRectValue], cellFrame)) {  
    SDImageCache *cache = [manager imageCache];
    NSString *key = [manager cacheKeyForURL:targetURL];
    if (![cache imageFromMemoryCacheForKey:key]) {
        shouldLoadImage = NO;
    }
}
if (shouldLoadImage) {  
    // load image
}
{% endhighlight %}

[[Ref.] UIScrollView 实践经验](https://tech.glowing.com/cn/practice-in-uiscrollview/)