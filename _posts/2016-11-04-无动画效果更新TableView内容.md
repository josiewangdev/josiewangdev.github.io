---
layout: post
title: 无动画效果更新TableView内容 · Update a TableView without animation
tags: iOS
categories: Dev
---

**Problem Description:**

When update a table view cell and you’re not scrolled at the top (or bottom if you’re inverted) of your table view, but you notice one of the follow behaviors:

* The UITableView scrolls to the top or bottom
* The UITableView looks hung over and flashses back and forth
* The UITableView jiggles just a bit but enough to notice

**Solution:**

1. Do the best you can establishing estimated heights for all of your table cell types. Even if heights are somewhat dynamic this helps the UITableView.

2. Save your scroll position and after updating your TableView and making a call to `endUpdates()` reset the content offset.

Here is the code required to maintain the scroll view’s content offset and prevent unnecessary animations from starting

**问题描述:**

当更新一个table view cell的内容并且没有滚动当前列表到最顶（或最底），会发现有以下几种情况出现：

* UITableView 滚动到了最顶或最底部
* UITableView 来回大幅闪动
* UITableView 小幅度闪烁了一下但是能够看出

**解决方法:**

1. 尽最大可能的估计你使用的所有类型的 table cell的大约高度，即使有些地方的table cell的高度使用的是动态高度。

2. 暂存列表的滚动位置，在完成TableView的更新之后，调用 `endUpdates()` 重置滚动视图的内容偏移值。

以下是无动画重置滚动视图内容偏移量的代码

---

{% highlight swift %}
let lastScrollOffset = tableView.contentOffset
tableView.beginUpdates()
...
tableView.endUpdates()
tableView.layer.removeAllAnimations()
tableView.setContentOffset(lastScrollOffset, animated: false)
{% endhighlight %}

