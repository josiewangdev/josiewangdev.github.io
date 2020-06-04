---
layout: post
title: DZNEmptyDataSet与MJRefresh的冲突 · Conflict of DZNEmptyDataSet and MJRefresh
tags: Bug
categories: Dev
---

**Problem Description:**

If a collection view use both `DZNEmptyDataSet` and `MJRefresh`, The empty data set view will offset the distance of a header.

**Solution:**

Set the content offset to zero in delegate `emptyDataSetWillAppear`.

**问题描述:**

在collection view中同时使用 `DZNEmptyDataSet` 和 `MJRefresh`时，`MJRefresh`会造成空数据页面错位一个header的高度。

**解决方法:**

在 `emptyDataSetWillAppear` 代理方法中将滚动视图的偏移量归零。

---

{% highlight objc %}
- (void)emptyDataSetWillAppear:(UIScrollView *)scrollView {
    scrollView.contentOffset = CGPointZero;
}
{% endhighlight %}

[[Ref.参考]DZNEmptyDataSet-Github](https://github.com/dzenbot/DZNEmptyDataSet)

[[Ref.参考]MJRefresh-Github](https://github.com/CoderMJLee/MJRefresh)
