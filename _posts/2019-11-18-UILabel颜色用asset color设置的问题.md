---
layout: post
title: UILabel颜色用asset color设置的问题 · The bug of setting UILabel text color using asset color
tags: Bug
categories: Dev	
description: "Xcode Bug"
---

**Bug Description :**

`UILabel` text color is not changing programmatically, if I set the text color in nib and define that color in assets.

**Solution :**

Set the text color in nib using the default color instead of the customized asset color, then change color programmatically will work fine.

**Xcode Version :**

10.2

**Bug 描述 :**

如果在项目中使用了color asset，并且在nib文件中 `UILabel` 文字的颜色使用的是选择asset设置的颜色，则用代码改变颜色无法生效。

**解决方法 :**

在nib文件中设置文字颜色使用默认的系统自带颜色，之后则可以正常使用代码改变设置的文字颜色。

**XCode 版本 :**

10.2

<hr>