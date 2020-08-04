---
layout: post
title: 口袋校园
tags: ObjC
categories: Projects
cover: https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/pocket_campus/pocketcampus.jpg
description: 2015.01 - 2015.07
---

## [口袋校园](https://itunes.apple.com/cn/app/kou-dai-xiao-yuan/id993705603?mt=8)  

---

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/pocket_campus/pocketcampus.jpg)

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

2.1 [使用 objc_setAssociatedObject 在运行时下为系统类添加属性](#p21)

2.2 [使用 Method Swizzling 在运行时重新映射方法](#p22)

2.3 [绘制当前视图截图](#p23)

2.4 [使用Eventkit事件库添加日历提醒事项](#p24)

---

<br>

### 1. 概述 {#p1}

---

口袋校园是我参与团队合作开发的一款为大学生提供服务的 App。

其中包括缴纳电费, 查询学生一卡通消费记录, 发布与查看校园招聘, 白条支付等功能。

---

<br>

> #### 1.1 基本信息 {#p11}

使用语言: **Objective-C**

系统要求: **iOS**

参与时间: **2015.01 - 2015.07**

负责部分:  

* 用户登陆、登出   
* 个人详情主页
* 校园招聘信息、通知中心
* 白条支付流程、账单详情
* 使用 Ping++ 集成支付宝、微信、银联等支付功能

<br>

> #### 1.2 结构解读 {#p12}

#### 主要框架

* M -- Models
    * Common models
    * Custom models

* V -- Views
    * Common views (like toast view, alert view, cells etc.)
    * Custom views

* C -- Controllers
    * Common controllers
    * Custom controllers

<br>

#### 网络层结构

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/pocket_campus/p12.jpg)

<br>

> #### 1.3 第三方框架的使用 {#p13}

* [SVProgressHUD - 创建prompt]()
* [MBProgressHUD - 创建prompt]()
* [MJRefresh - 列表上拉刷新，下拉加载]()
* [Masonry - 使用纯代码方案解决屏幕大小自适应问题]()
* [XLForm - 创建动态表单的框架]()
* [AFNetworking - 网络层]()
* [AFNetworkActivityLogger - 使用AFN时的调试打印工具]()
* [SDWebImage - 网络加载图片]()
* [MagicalRecord - 使用Core Data做数据库存储的解决方案]()
* [MJExtension - 轻量级的模型与字典、数组等数据结构的转换]()
* Pingpp/Alipay - Ping ++ 支付宝组件
* Pingpp/Wx - Ping ++ 微信支付组件
* Pingpp/UnionPay - Ping ++ 银联组件

---

<br>

### 2. 开发随笔 {#p2}

---

<br>

> #### 2.1 使用 objc_setAssociatedObject 在运行时下为系统类关联属性 {#p21}

在 OS X 10.6 之后，Runtime系统让ObjC支持向对象动态添加变量。

涉及到的函数有以下三个, 这些方法以键值对的形式动态地向对象添加、获取或删除关联值。

```c
//  添加
void objc_setAssociatedObject(idobject,constvoid*key, idvalue, objc_AssociationPolicy policy);
//  获取
id objc_getAssociatedObject(idobject,constvoid*key);
//  删除
void objc_removeAssociatedObjects(idobject);
```

其中 `objc_AssociationPolicy` 关联政策是一组枚举常量：

```objc
OBJC_ASSOCIATION_ASSIGN = 0,          <指定一个弱引用关联的对象>
OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,<指定一个强引用关联的对象>
OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  <指定相关的对象复制>
OBJC_ASSOCIATION_RETAIN = 01401,      <指定强参考>
OBJC_ASSOCIATION_COPY = 01403         <指定相关的对象复制>
```

有了这些方法, 可以轻松的解决我们在无法更改某个类源代码的情况下, 却向这个类中添加(实际上是关联)一些自定义的 **property** 。

例如给系统类 **UITextView** 关联 **placeHolder** 与 **placeHolderLabel** 属性 。

```objc
//  UITextView+PlaceHolder.h
@interface UITextView (PlaceHolder)

@property (nonatomic) NSAttributedString *placeHolder;
@property (nonatomic) UILabel *placeHolderLabel;

@end
```

```objc
//  UITextView+PlaceHolder.m
#import <objc/runtime.h>

@implementation UITextView (PlaceHolder)

@dynamic placeHolder;
@dynamic placeHolderLabel;

- (void)setPlaceHolder:(NSAttributedString *)ph {
    if (self.placeHolderLabel == nil) {

        UILabel *placeHolderLabel = [UILabel new];
        placeHolderLabel.textColor = [UIColor lightGrayColor];
        self.placeHolderLabel = placeHolderLabel;
        [self addSubview:placeHolderLabel];
        
        [placeHolderLabel makeConstraints:^(MASConstraintMaker *make) {
            make.top.equalTo(self).offset(7);
            make.left.equalTo(self).offset(7);
        }];

    }

    self.placeHolderLabel.attributedText = ph;
    objc_setAssociatedObject(self, @selector(placeHolder), ph, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSString *)placeHolder {
    return objc_getAssociatedObject(self, @selector(placeHolder));
}

- (void)setPlaceHolderLabel:(UILabel *)phl {
    objc_setAssociatedObject(self, @selector(placeHolderLabel), phl, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (UILabel *)placeHolderLabel {
    return objc_getAssociatedObject(self, @selector(placeHolderLabel));
}
@end
```

再使用 **UITextView** 时就可以直接调用

```objc
textView.placeHolder = @"my place holder";
textView.placeHolderLabel.hidden = YES;
```

<br>

> #### 2.2 使用 Method Swizzling 在运行时重新映射方法 {#p22}

使用 Method Swizzling 将系统 **viewDidLoad** 方法映射为我的 **my_viewDidLoad** 方法, 之后可以观察到每当 ViewController 被创建时, 都打印了 **"did load XXX"** 信息, 即方法已经被替换执行了。

```objc
//  UIViewController+Swizzle.m
#import <objc/runtime.h>

@implementation UIViewController (Swizzle)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method originalMethod = class_getInstanceMethod([self class], @selector(viewDidLoad));
        Method targetMethod = class_getInstanceMethod([self class], @selector(my_viewDidLoad));
        method_exchangeImplementations(originalMethod, targetMethod);
    });
}

- (void)my_viewDidLoad {
    [self my_viewDidLoad];
    NSLog(@"%@ did load", NSStringFromClass([self class]));
}
```

<br>

> #### 2.3 绘制当前视图截图 {#p23}

```swift
- (UIImage *)capture {
    UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, self.view.opaque, 0.0);
    [[UIApplication sharedApplication].keyWindow.layer renderInContext:UIGraphicsGetCurrentContext()];
    UIImage * img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return img;
}
```

<br>

> #### 2.4 使用Eventkit事件库添加日历提醒事项 {#p24}

事件提醒开发包（EventKit）由事件库、事件源、日历和事件/提醒组成，他们的关系是：事件库用于直接操作日历数据库，日历数据库中的数据按事件源、日历和事件/提醒三级进行分类组织。

每个事件源对应一个准帐户，该帐户下可以有多个日历，日历分两类，一类是用于存储事件的日历，一类是用于存储提醒的日历。

事件库框架授权访问用户的 Calendar.app 和 Reminders.app 应用的信息。尽管是用两个不同的应用显示用户的日历和提醒数据，但确是同一个框架维护这份数据。同样地，存储这份数据的数据库叫做日历数据库，同时容纳日历和提醒信息。 

事件库不但允许你的应用获取用户已经存在的日历及提醒数据，而且它可以让你的应用为任何日历创建新的事件和提醒。另外，事件库让用户可以编辑和删除他们的事件和提醒。更高级的任务，诸如添加闹钟或指定循环事件，也可以使用事件库完成。如果日历数据库有来自你的应用外部的更改发生，事件库可以通过通知监测到，这样你的应用可以做出适当的响应。使用事件库对日历项所做的更改会自动地同步到相关的日历。

<br>

**创建事件并添加到提醒事项**

```objc
//  CalendarAction.h

@interface CalendarAction : NSObject
+  (void)saveEventWithStartDate:(NSDate*)startDate
                        endDate:(NSDate*)endDate
                          alarm:(NSTimeInterval)timeOffset
                     eventTitle:(NSString*)title
                  eventLocation:(NSString*)location
                    notes:(NSString *)notes
                     isReminder:(BOOL)isReminder;
@end
```

```objc
//  CalendarAction.m
#import <EventKit/EventKit.h>

@implementation CalendarAction

+  (void)saveEventWithStartDate:(NSDate*)startDate
                    endDate:(NSDate*)endDate
                      alarm:(NSTimeInterval)timeOffset
                 eventTitle:(NSString*)title
                  eventLocation:(NSString*)location
                    notes:(NSString *)notes
                 isReminder:(BOOL)isReminder{
    
    EKEventStore *eventStore = [[EKEventStore alloc] init];
    
    if ([eventStore respondsToSelector:@selector(requestAccessToEntityType:completion:)]) {
        [eventStore requestAccessToEntityType:EKEntityTypeEvent completion:^(BOOL granted, NSError *error) {
            //  添加到UI的主queue中，否则不显示HUD
            dispatch_async(dispatch_get_main_queue(), ^{
                if (granted) {
                    [self createEventWithStartDate:startDate endDate:endDate alarm:timeOffset eventTitle:title eventLocation:location notes:notes eventStore:eventStore];
                    
                    if (isReminder) {
                        [self createReminderWithStartDate:startDate endDate:endDate alarm:timeOffset eventTitle:title eventLocation:location notes:notes eventStore:eventStore];
                    }
                }
            });
        }];
    }else{
        [self createEventWithStartDate:startDate endDate:endDate alarm:timeOffset eventTitle:title eventLocation:location notes:notes eventStore:eventStore];

        if (isReminder) {
            [self createReminderWithStartDate:startDate endDate:endDate alarm:timeOffset eventTitle:title eventLocation:location notes:notes eventStore:eventStore];
        }
    }
}

+ (void)createEventWithStartDate:(NSDate*)startDate
                        endDate:(NSDate*)endDate
                          alarm:(NSTimeInterval)timeOffset
                     eventTitle:(NSString*)title
                    eventLocation:(NSString*)location
                     notes:(NSString *)notes
                       eventStore:(EKEventStore *)eventStore{
    
    EKEvent *event = [EKEvent eventWithEventStore:eventStore];
    event.title = title;
    event.location = location;
    event.notes = notes;
    event.startDate = startDate;
    event.endDate = endDate;
    event.allDay = YES;
    event.calendar = [eventStore defaultCalendarForNewEvents];
    [event addAlarm:[EKAlarm alarmWithRelativeOffset:timeOffset]];
    NSError *error = nil;
    [eventStore saveEvent:event span:EKSpanThisEvent error:&error];
    if (error) {
        [SVProgressHUD showErrorWithStatus:error.localizedDescription];
    }else{
        [SVProgressHUD showSuccessWithStatus:NSLocalizedString(@"成功添加日历", nil)];
    }
}

+ (void)createReminderWithStartDate:(NSDate*)startDate
                         endDate:(NSDate*)endDate
                           alarm:(NSTimeInterval)timeOffset
                      eventTitle:(NSString*)title
                   eventLocation:(NSString*)location
                        notes:(NSString *)notes
                      eventStore:(EKEventStore *)eventStore{
    
    EKReminder *reminder = [EKReminder reminderWithEventStore:eventStore];
    reminder.title = title;
    reminder.location = location;
    reminder.notes = notes;
    reminder.calendar = [eventStore defaultCalendarForNewReminders];
    [reminder addAlarm:[EKAlarm alarmWithRelativeOffset:timeOffset]];
    NSError *error = nil;
    [eventStore saveReminder:reminder commit:YES error:&error];
    if (error) {
        [SVProgressHUD showErrorWithStatus:error.localizedDescription];
    }
}
@end

```

**使用时直接调用**


```objc
[CalendarAction saveEventWithStartDate:startDate 
                                endDate:endDate
                                alarm:alarm 
                                eventTitle:eventTitle 
                                eventLocation:eventLocation 
                                notes:nil 
                                isReminder:NO];
```