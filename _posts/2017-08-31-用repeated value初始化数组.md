---
layout: post
title: 用repeated value初始化数组 · Init array using repeated value
tags: iOS
categories: Dev
---

When you use the function `init(count:, repeatedValue:)` to initialize an array, you should know each value of this array references the same instance. So if you change anyone of them, all the others ones would also be changed.

如果你使用了 `init(count:, repeatedValue:)` 方法进行数组初始化，则你需要知道这个数组是创建了多个指向同一个内存地址的指针，所以当你改变了数组中任意的一个值时，其他所有的值都会被改变。

---

{% highlight swift %}
let array = [AnyObject](count: count, repeatedValue: AnyObject().init())
{% endhighlight %}