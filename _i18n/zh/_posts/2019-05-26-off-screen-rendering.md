---
layout: post
title: 离屏渲染
tags: iOS
categories: Dev
---

## 离屏渲染

---

#### 1. [GPU屏幕渲染的两种方式](#p1)
#### 2. [为什么要使用离屏渲染？](#p2)
#### 3. [离屏渲染的过程](#p3)
#### 4. [界面卡顿的原因](#p4)
#### 5. [会引发离屏渲染的操作](#p5)
#### 6. [圆角属性优化方案](#p6)
#### 7. [阴影优化方案](#p7)

---

<br>

> #### 1. GPU屏幕渲染的两种方式 {#p1}

**当前屏幕渲染**

指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区进行。当前屏幕渲染不需要额外创建新的缓存，也不需要开启新的上下文，相对于离屏渲染性能更好。

**离屏渲染**

指的是在GPU在当前屏幕缓冲区以外开辟一个缓冲区进行渲染操作。相比于当前屏幕渲染，离屏渲染的代价是很高的。

---

<br>

> #### 2. 为什么要使用离屏渲染？ {#p2}

当有些图层的属性混合效果受自身上下文，屏幕缓存等限制因素，不能直接绘制于屏幕上，需要在别的地方做额外的处理预合成，就使用到离屏渲染。意味着图层必须在被显示之前在一个屏幕外上下文中被渲染。

---

<br>

> #### 3. 离屏渲染的过程 {#p3}

**1).** 创建一个新的缓冲区

**2).** 多次切换上下文环境

---

<br>

> #### 4. 界面卡顿的原因 {#p4}

离屏渲染先是从当前屏幕（On-screen）切换到离屏（Off-screen），等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上有需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价。由于垂直同步的机制，如果在一个显示器的刷新时间内，GPU没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

---

<br>

> #### 5. 会引发离屏渲染的操作 {#p5}
 
**1).** 为图层设置遮罩（layer.mask), 将图层的layer.masksToBounds / view.clipsToBounds属性设置为true

**2).** 任何以CG开头的class

**3).** 将图层layer.allowsGroupOpacity属性设置为YES和layer.opacity小于1.0

**4).** 为图层设置阴影（layer.shadow *）

**5).** 为图层设置layer.shouldRasterize=true

**6).** 使用CGContext在drawRect :方法中绘制大部分情况下会导致离屏渲染，甚至仅仅是一个空的实现。

**7).** 文本（任何种类，包括UILabel，CATextLayer，Core Text等）

---

<br>

> #### 6. 圆角属性优化方案 {#p6}

同时设置 `layer.cornerRadius` 和 `layer.masksToBounds` 会发生离屏渲染。

例如：

```swift
//  Set the rounded corners of the UIView - will not produce off-screen rendering
self.view.layer.cornerRadius = 10;
 
//  Set the rounded corners of the UIButton - generate off-screen rendering
self.btn.layer.cornerRadius = 20;
self.btn.layer.masksToBounds = YES;
    
//  Set the rounded corners of the UIImageView - generate off-screen rendering
self.imageView.layer.cornerRadius = 30;
self.imageView.layer.masksToBounds = YES;
```

优化方法：

```swift
//  1. Load image
UIImage * image = [UIImage imageNamed:@"1.png"];
 
//  2. Turn on bitmap context
UIGraphicsBeginImageContext(image.size);
    
//  3. Set a circular clipping area
UIBezierPath * path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(0, 0, image.size.width, image.size.height) cornerRadius:image.size.width];
// Set the path to the crop area. Content beyond the crop area will be automatically cropped (the effect on the content drawn later, the context content already drawn will not be cropped)
[path addClip];
    
//  4. Draw the image into the context
[image drawAtPoint:CGPointZero];
    
//  5. Generate an image from the context
UIImage * newImage =  UIGraphicsGetImageFromCurrentImageContext();
    
//  6. Close the context
UIGraphicsEndImageContext();
    
self.imageView.image = newImage;
```

---

<br>

> #### 7. 阴影优化方案 {#p7}

使用 `ShadowPath` 而非设置 `shadowOffset` 等属性。

```swift
self.imageView.layer.shadowColor = [UIColor grayColor].CGColor;
self.imageView.layer.shadowOpacity = 1.0;
self.imageView.layer.shadowRadius = 2.0;
self.imageView.layer.shadowOffset = CGSizeMake(10, 10);
  
//  Set the path of the shadow
UIBezierPath *path = [UIBezierPath bezierPathWithRect:self.imageView.bounds];
self.imageView.layer.shadowPath = path.CGPath;
```

---

<br>

> #### 参考链接

* [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)