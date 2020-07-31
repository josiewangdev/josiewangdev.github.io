---
layout: post
title: Off-screen Rendering
tags: iOS
categories: Dev
---

## Off-screen Rendering

---

#### 1. [GPU屏幕渲染的两种方式](#p1)
#### 2. [Why off-screen rendering?](#p2)
#### 3. [离屏渲染的过程](#p3)
#### 4. [界面卡顿的原因](#p4)
#### 5. [Operations lead to off-screen rendering](#p5)
#### 6. [Fillet attributes optimization](#p6)
#### 7. [Shadow optimization](#p7)

---

<br>

> #### 1. GPU屏幕渲染的两种方式 {#p1}

**On-screen Rendering**

指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区进行。当前屏幕渲染不需要额外创建新的缓存，也不需要开启新的上下文，相对于离屏渲染性能更好。

**Off-screen Rendering**

指的是在GPU在当前屏幕缓冲区以外开辟一个缓冲区进行渲染操作。相比于当前屏幕渲染，离屏渲染的代价是很高的。

---

<br>

> #### 2. Why off-screen rendering? {#p2}

A mixture of layer properties cannot be drawn directly on the screen without pre-synthesis, so off-screen rendering is required.

#### 为什么要使用离屏渲染？####

但有些图层的属性混合效果受自身上下文，屏幕缓存等限制因素，不能直接绘制于屏幕上，需要在别的地方做额外的处理预合成，就使用到离屏渲染。意味着图层必须在被显示之前在一个屏幕外上下文中被渲染。

---

<br>

> #### 3. 离屏渲染的过程 {#p3}

**1).** 创建一个新的缓冲区。

**2).** 多次切换上下文环境。

---

<br>

> #### 4. 界面卡顿的原因 {#p4}

先是从当前屏幕（On-screen）切换到离屏（Off-screen），等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上有需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价。由于垂直同步的机制，如果在一个显示器的刷新时间内，GPU没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

---

<br>

> #### 5. Operations lead to off-screen rendering {#p5}

**1).** layer mask settings

**2).** Core Graphics (any class prefixed with CG)

**3).** Group opacity (UIViewGroupOpacity).

**4).** CALayers using masks (setMasksToBounds) and dynamic shadows (setShadow).

**5).** CALayers with a shouldRasterize property set to YES.

**6).** The drawRect() method, even with an empty implementation.

**7).** Any text displayed on screen, including Core Text.

---

<br>

> #### 6. Fillet attributes optimization {#p6}

Setting `layer.cornerRadius` and `layer.masksToBounds` at the same time will produce off-screen rendering.


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

> #### 7. Shadow optimization {#p7}

Use ShadowPath instead of the settings of properties such as `shadowOffset`.

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

> #### Reference links

* [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

