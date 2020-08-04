---
layout: post
title: 数字报道
tags: ObjC
categories: Projects
cover: https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/campuscloud.jpg
description: 2015.12 - 2016.03
---

## [数字报道](https://itunes.apple.com/cn/app/%E7%BE%9A%E8%B7%91%E4%BD%93%E8%82%B2-%E5%AE%B6%E6%A0%A1%E4%BA%92%E5%8A%A8-%E5%85%B1%E5%90%8C%E6%88%90%E9%95%BF/id1233542475?mt=8)

---

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/campuscloud.jpg)

---

<br>

### 目录

---

#### 1. [概述](#p1)

1.1 [基本信息](#p11)

1.2 [结构解读](#p12)

1.3 [第三方框架的使用](#p13)

---

#### 2. [开发随笔](#p2)

2.1 [复杂需求逻辑与分析](#p21)

2.2 [水波纹背景效果实现](#p22)

2.3 [多列表的联动效果](#p23)

---

<br>

### 1. 概述 {#p1}

---

数字报道是为移动通信运营商与大学校园合作而催生的一款手机端应用。 旨在学生享受在线网上报道功能的同时, 选择合作的移动运营商提供的手机号码与业务套餐。

整个 App 提供的报道功能里包括在线选择宿舍, 购买军训用品, 录入报道信息, 登记到校时间等。

---

<br>

> #### 1.1 基本信息 {#p11}

使用语言: **Objective-C**

系统要求: **iOS**

参与时间: **2015.12 - 2016.03**

负责部分: 

* 自助报道模块
* 校园招聘模块
* 用户登陆、登出   

<br>

> #### 1.2 结构解读 {#p12}

#### 网络层结构

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/p12.jpg)

<br>

> #### 1.3 第三方框架的使用 {#p13}

* [DZNPhotoPickerController - 相册图片选择、剪裁编辑器]()
* [SVProgressHUD - 创建prompt]()
* [SWRevealViewController - 侧边栏菜单]()
* [MJRefresh - 列表上拉刷新，下拉加载]()
* [Masonry - 使用纯代码方案解决屏幕大小自适应问题]()
* [XLForm - 创建动态表单的框架]()
* [AFNetworking - 网络层]()
* [AFNetworkActivityLogger - 使用AFN时的调试打印工具]()
* [SDWebImage - 网络加载图片]()
* [MJExtension - 轻量级的模型与字典、数组等数据结构的转换]()
* [UICollectionViewLeftAlignedLayout - UICollection View 居左自动布局]()
* [BlocksKit - 系统方法与组件的一些block封装]()
* [UMengFeedback - 友盟反馈]()
* [UMengAnalytics-NO-IDFA - 友盟统计]()

---

<br>

### 2. 开发随笔 {#p2}

---

<br>

> #### 2.1 复杂需求逻辑与分析 {#p21}

这款应用因与多家高校合作, 但不同的高校有不同的报道流程, 所以需要根据每所学校在后台配置的流程来展示不同的报道流程页面。

所以在前端实现时, 所有的页面顺序是不固定的, 相比固定流程的页面, 需求复杂度就大幅上升了。

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/dynamic_flow.jpg)

根据需求, 大概确定开放框架。

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/hierarchy.jpg)

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/p21_1.jpg) | ![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/p21_2.jpg)

<br>

> #### 2. 水波纹背景效果实现 {#p22}

项目中, 有个需求为下图中所示的水波纹效果作为背景, 并且需要有动态化的动画效果, 因此不能使用UI给的静态图片直接作为背景, 而是使用代码绘制。

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/water_wave.jpg)

```objc
//  WaterWaveView.h

@interface WaterWaveView : UIView
- (instancetype)initWithWaterWaveColor:(UIColor *)color;
@end
```

```objc
@interface WaterWaveView()

@property (nonatomic, strong) UIColor *waveColor;
@property (nonatomic, assign) CGFloat paraA;
@property (nonatomic, assign) CGFloat paraB;
@property (nonatomic, assign) CGFloat startY;
@property (nonatomic, assign) BOOL reverse;

@end

@implementation WaterWaveView

- (instancetype)initWithWaterWaveColor:(UIColor *)color{
    self = [super init];
    if (self) {
        self.waveColor = color;
        self.backgroundColor = [UIColor clearColor];
        self.paraA = 0.8;
        self.paraB = 0;
        self.reverse = NO;
        [NSTimer scheduledTimerWithTimeInterval:0.02 target:self selector:@selector(createWaveMotion) userInfo:nil repeats:YES];
    }
    return self;
}

- (void)layoutSubviews{
    [super layoutSubviews];
    self.startY = self.bounds.size.height/2;
}

- (void)createWaveMotion{
    if (self.reverse) {
        self.paraA -= 0.01;
    }else{
        self.paraB += 0.01;
    }
    
    if (self.paraA >= 1) {
        self.reverse = YES;
    }else if (self.paraA <= 0.3){
        self.reverse = NO;
    }
    
    self.paraB += 0.03;
    [self setNeedsDisplay];
}

// Only override drawRect: if you perform custom drawing.
// An empty implementation adversely affects performance during animation.

- (void)drawRect:(CGRect)rect {
    [self drawWaterWaveWithColor:self.waveColor alpha:0.14 waveHeightParameter:self.paraA*3 waveLengthParameter:self.paraB inRect:rect];
    [self drawWaterWaveWithColor:self.waveColor alpha:0.08 waveHeightParameter:self.paraA waveLengthParameter:self.paraB*3 inRect:rect];
}

- (void)drawWaterWaveWithColor:(UIColor *)color
                         alpha:(CGFloat)alpha
           waveHeightParameter:(CGFloat)paraHeight
           waveLengthParameter:(CGFloat)paraLength
                         inRect:(CGRect)rect{
    
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGMutablePathRef path = CGPathCreateMutable();
    
    CGContextSetLineWidth(context, 1);
    CGContextSetFillColorWithColor(context, color.CGColor);
    CGContextSetAlpha(context, alpha);
    
    float y = self.startY;
    CGPathMoveToPoint(path, nil, 0, y);
    for(float x = 0; x <= DEVICE_WIDTH; x++){
        y = paraHeight * sin( x * M_PI/180 + 2 * paraLength/M_PI ) * 5 + self.startY;
        CGPathAddLineToPoint(path, nil, x, y);
    }
    
    CGPathAddLineToPoint(path, nil, DEVICE_WIDTH, rect.size.height);
    CGPathAddLineToPoint(path, nil, 0, rect.size.height);
    CGPathAddLineToPoint(path, nil, 0, self.startY);
    
    CGContextAddPath(context, path);
    CGContextFillPath(context);
    CGContextDrawPath(context, kCGPathStroke);
    CGPathRelease(path);
    
}

@end
```

<br>

> #### 3. 多列表的联动效果 {#p23}

如图所示, 在 **列表2** 滚动时, 需要实现 **列表1** 联动的效果。

<br>

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/new_campus/p23.jpg)

<br>

这个需求实现的关键点主要在于滚动视图代理方法中, 正确处理UI的变化。

```objc
#pragma mark - UIScrollViewDelegate

- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    if (scrollView.tag == TagDetail && self.isManualScroll) {
        NSInteger visibleDetailsSection = [self.detailsTableView indexPathForCell:self.detailsTableView.visibleCells[0]].section;
        NSIndexPath *selectedIndexPath = [NSIndexPath indexPathForRow:visibleDetailsSection inSection:0];
        [self.categoryTableView selectRowAtIndexPath:selectedIndexPath animated:YES scrollPosition:UITableViewScrollPositionNone];
    }
}

- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    self.isManualScroll = YES;
}

- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{
    if (!decelerate) {
        self.isManualScroll = NO;
    }
}
```
