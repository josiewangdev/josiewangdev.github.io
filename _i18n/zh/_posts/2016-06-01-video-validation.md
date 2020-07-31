---
layout: post
title: 快审
tags: ObjC
categories: Projects
cover: images/apps/kuaishen.jpg
description: 2016.07 - 2017.01
---

## [快审](https://itunes.apple.com/cn/app/kuai-shen/id1138598976?mt=8)  

---

![img](/images/apps/kuaishen.jpg)

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

2.1 [键盘弹出时列表向上移动](#p21)

2.2 [扫描二维码](#p22)

2.3 [身份证号验证](#p23)

---

<br>

### 1. 概述 {#p1}

---

快审是一款用于本地电信营业厅视频审核新购号码用户的 App。 

整个 App 包括扫码排队, 视频审核, 资料上传提交, 新购订单查询等功能。

实时音视频对话模块组件是基于网易云信服务实现。 

---

<br>

> #### 1.1 基本信息 {#p11}

使用语言: **Objective-C**

系统要求: **iOS**

参与时间: **2016.07 - 2017.01**

负责部分: **个人独立开发所有模块**

<br>

> #### 1.2 结构解读 {#p12}

#### 主要框架

* M -- Models
    * Common models
    * Custom models

* Modules 
    * Module 1
        * V - Views
        * C - Controllers

    * Module 2
        * V - Views
        * C - Controllers

    * Module 2
        * V - Views
        * C - Controllers
        
    ... ...

<br>

#### 网络层结构

![img](/images/video_validation/p12.jpg)

<br>

> #### 1.3 第三方框架的使用 {#p13}

* [DZNPhotoPickerController - 相册图片选择、剪裁编辑器]()
* [SVProgressHUD - 创建prompt]()
* [MJRefresh - 列表上拉刷新，下拉加载]()
* [Masonry - 使用纯代码方案解决屏幕大小自适应问题]()
* [XLForm - 创建动态表单的框架]()
* [AFNetworking - 网络层]()
* [AFNetworkActivityLogger - 使用AFN时的调试打印工具]()
* [SDWebImage - 网络加载图片]()
* [MJExtension - 轻量级的模型与字典、数组等数据结构的转换]()
* [LBXScan - 二维码、条形码识别]()
* [BlocksKit - 系统方法与组件的一些block封装]()
* [NIMSDK - 网易云信音视频对话组件]()
* [Bugly - 崩溃监测统计]()

---

<br>

### 2. 开发随笔 {#p2}

---

<br>

> #### 2.1 键盘弹出时列表向上移动 {#p21}

```objc
- (void)viewDidLoad {
    //Add keyboard observer
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillShowNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillHide:) name:UIKeyboardWillHideNotification object:nil];
}

- (void)keyboardWillShow:(NSNotification *)notification{
    [self.tableView updateConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.topLogoImageView.bottom).offset(-self.view.bounds.size.height * 0.21);
    }];
    
    [UIView animateWithDuration:0.5 delay:0 usingSpringWithDamping:0.7 initialSpringVelocity:1.0 options:kNilOptions animations:^{
        [self.tableView layoutIfNeeded];
    } completion:nil];
}

- (void)keyboardWillHide:(NSNotification *)notification{
    [self.tableView updateConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.topLogoImageView.bottom).offset(-kTableViewTopInset);
    }];
    
    [UIView animateWithDuration:0.5 delay:0 usingSpringWithDamping:0.7 initialSpringVelocity:1.0 options:kNilOptions animations:^{
        [self.tableView layoutIfNeeded];
    } completion:nil];
}
```

<br>

> #### 2.2 扫描二维码 {#p22}

```objc
//  ScanCodeViewController.h

#import <LBXScan/LBXScanViewController.h>

@interface ScanCodeViewController : LBXScanViewController

@end
```

```objc
//  ScanCodeViewController.m

- (void)scanCode {
    LBXScanViewStyle *style = [[LBXScanViewStyle alloc] init];
    //矩形区域中心上移，默认中心点为屏幕中心点
    style.centerUpOffset = 0;
    style.xScanRetangleOffset = 40;
    style.isNeedShowRetangle = NO;
    
    style.photoframeAngleStyle = LBXScanViewPhotoframeAngleStyle_Outer;
    style.photoframeAngleW = 24;
    style.photoframeAngleH = 24;
    style.photoframeLineW = 4;
    
    style.anmiationStyle = LBXScanViewAnimationStyle_NetGrid;
    style.animationImage = [UIImage imageNamed:@"scan"];
    
    ScanCodeViewController *scanCodeVC = [[ScanCodeViewController alloc] init];
    scanCodeVC.style = style;
    [self.navigationController pushViewController:scanCodeVC animated:NO];
}
```

<br>

> #### 2.3 身份证号验证 {#p23}

```objc
//  身份证号验证

+ (BOOL)checkIsIdCardNumber:(NSString *)idNumber{
    if (idNumber.length == 15) {
        
        NSString *idCard15Regex = @"^[1-9]\\d{7}((0\\d)|(1[0-2]))(([0|1|2]\\d)|3[0-1])\\d{3}$";
        NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", idCard15Regex];
        if (![predicate evaluateWithObject:idNumber]) {
            return NO;
        }
        
        //将15位身份证转为18位验证
        return [self checkIs18BitIdCardNumber:[self convert15IdCardTo18Bit:idNumber]];
        
    }else if (idNumber.length == 18){

        NSString *idCard18Regex  = @"^[1-9]\\d{5}[1-9]\\d{3}((0\\d)|(1[0-2]))(([0|1|2]\\d)|3[0-1])\\d{3}([\\d|x|X]{1})$";
        NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", idCard18Regex];
        if (![predicate evaluateWithObject:idNumber]) {
            return NO;
        }

        return [self checkIs18BitIdCardNumber:idNumber];
        
    }else{
        return NO;
    }
}


//  将15位的身份证转成18位身份证

+ (NSString *)convert15IdCardTo18Bit:(NSString *)idNumber15{
    NSString *newBirthStr = [@"19" stringByAppendingString:[idNumber15 substringWithRange:NSMakeRange(6, 6)]];
    NSString *idNumber17 = [idNumber15 stringByReplacingCharactersInRange:NSMakeRange(6, 6) withString:newBirthStr];

    NSArray *powerFactors = @[ @"7", @"9", @"10", @"5", @"8", @"4", @"2", @"1", @"6", @"3", @"7", @"9", @"10", @"5", @"8", @"4", @"2" ];
    NSArray *verifyCodes = @[ @"1", @"0", @"X", @"9", @"8", @"7", @"6", @"5", @"4", @"3", @"2" ];
    NSInteger powerSum = 0;
    for (int i = 0; i < 17; i++) {
        powerSum += [[idNumber17 substringWithRange:NSMakeRange(i,1)] intValue] * [powerFactors[i] intValue];
    }

    NSInteger verifyCodeIndex = powerSum%11;
    NSString *idNumber18 = [idNumber17 stringByAppendingString:verifyCodes[verifyCodeIndex]];
    
    return idNumber18;   
}


//  18位身份证号码校验

+ (BOOL)checkIs18BitIdCardNumber:(NSString *)idNumber18{
    //判断省份代码
    NSString *provinceCode = [idNumber18 substringToIndex:2];
    NSArray *cityCodeArray = @[@"11", @"12", @"13", @"14", @"15", @"21", @"22",@"23", @"31",
                               @"32", @"33", @"34", @"35", @"36", @"37", @"41", @"42", @"43",@"44",
                               @"45", @"46", @"50", @"51", @"52", @"53", @"54", @"61", @"62", @"63",
                               @"64", @"65", @"71", @"81", @"82", @"91"];
    if (![cityCodeArray containsObject:provinceCode]) {
        return NO;
    }
    
    //判断生日是否合法
    NSString *birthStr = [idNumber18 substringWithRange:NSMakeRange(6, 8)];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    formatter.dateFormat = @"yyyyMMdd";
    if([formatter dateFromString:birthStr] == nil){
        return NO;
    }
    
    //判断校验位
    //前17位加权因子
    NSArray *powerFactors = @[@"7", @"9", @"10", @"5", @"8", @"4", @"2", @"1", @"6", @"3", @"7", @"9", @"10", @"5", @"8", @"4", @"2"];
    // 除以11后，可能产生的11位余数、第18位校检码
    NSArray *verifyCodes = @[@"1", @"0", @"X", @"9", @"8", @"7", @"6", @"5", @"4", @"3", @"2"];
    //加权因子总和
    NSInteger powerSum = 0;
    for (int i = 0; i < 17; i++) {
        powerSum += [[idNumber18 substringWithRange:NSMakeRange(i,1)] intValue] * [powerFactors[i] intValue];
    }
    //计算出校验码所在数组的位置
    NSInteger verifyCodeIndex = powerSum%11;
    //计算出校验码所在数组的位置
    NSString *lastNumber = [idNumber18 substringWithRange:NSMakeRange(17,1)];
    if ([lastNumber caseInsensitiveCompare:verifyCodes[verifyCodeIndex]] != NSOrderedSame) {
        return NO;
    }
    
    return YES;
}
```
