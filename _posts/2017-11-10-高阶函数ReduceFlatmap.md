---
layout: post
title: 高阶函数 - Reduce, Flatmap · Higher order fuctions - Reduce, Flatmap
tags: Swift
categories: Dev
---

#### Reduce ####

---

Use reduce to combine all items in a collection to create a single new value.

使用reduce来组合集合中的所有元素并返回一个新的非集合类型的值。

---

{% highlight swift %}
let items = [2.0,4.0,5.0,7.0]
let total = items.reduce(10.0,combine: +)
// 28.0

let items = [2.0,4.0,5.0,7.0]
let total = items.reduce(10.0,+)
// 28.0
{% endhighlight %}

{% highlight swift %}
let codes = ["abc","def","ghi"]
let text = codes.reduce("", combine: +)
// "abcdefghi"

let codes = ["abc","def","ghi"]
let text = codes.reduce("",+)
// "abcdefghi"
{% endhighlight %}

---

#### FlatMap ####

---

Flatmap is used to flatten a collection of collections.

Flatmap用于将一个二维数组拆开展平.

---

{% highlight swift %}
let collections = [[5,2,7],[4,8],[9,1,3]]
let flat = collections.flatMap { $0 }
// [5, 2, 7, 4, 8, 9, 1, 3]
{% endhighlight %}

---

Flatmap is also used for removing nil values from a collection. 

Flatmap 也可以用来判断集合中的空值，并将空值移出集合。

---

{% highlight swift %}
let people: [String?] = ["Tom",nil,"Peter",nil,"Harry"]
let valid = people.flatMap {$0}
// ["Tom", "Peter", "Harry"]
{% endhighlight %}


