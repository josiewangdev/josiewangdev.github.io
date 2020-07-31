---
layout: post
title: Objective-C 随笔
tags: Notes
categories: Dev
---

## Objective-C 随笔

---

#### 1. [单例](#singleton)
#### 2. [__block关键字修饰变量](#block)
#### 3. [Predicate 查询语句](#predicate)
#### 4. [数字转换为人民币格式](#p4)
#### 5. [十进制数字的四舍五入](#p5)
#### 6. [md5加密字符串](#p6)
#### 7. [16进制色值转颜色](#p7)
#### 8. [颜色变暗](#p8)

---

<br>

> #### 1. 单例 {#singleton}

```objc
+ (instancetype)sharedInstance {
    static ClassName *sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[ClassName alloc] init];
    });
    return sharedInstance;
}
```

---

<br>

> #### 2. __block关键字修饰变量 {#block}

通常情况下，在block中的变量默认是将外部变量复制到其数据结构中来访问实现的，在block中对变量的任何修改不会影响到外部变量。

![block](/images/block/block2.png)

```objc
char localCharacter;

void (^aBlock)(void) = ^(void) {
    int c;
    c = localCharacter; // use localCharacter value but cannot change it
};
```

当使用 `__block` 关键字来修饰的外部变量时，在block中对变量的改变则会同样影响到外部变量，是因为在block中将不再是访问该外部变量复制的值，而是直接访问了该外部变量。

![block](/images/block/block1.png)

```objc
__block char localCharacter;

void (^aBlock)(void) = ^(void) {
    localCharacter = 'a'; // could change localCharacter value in enclosing scope
};
```

---

<br>

> #### 3. Predicate 查询语句 {#predicate}

OC中使用 `NSPredicate` 的方法

```objc
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name contains[c] %@", searchText];
```

---

<br>

> #### 4. 数字转换为人民币格式 {#p4}

```objc
@interface NSNumberFormatter (Helper)
+ (NSNumberFormatter *)RMBFormatter;
@end
```

```objc
@implementation NSNumberFormatter (Helper)
+ (NSNumberFormatter *)RMBFormatter {
    NSNumberFormatter *formatter = [NSNumberFormatter new];
    formatter.locale = [NSLocale localeWithLocaleIdentifier:@"zh_CN"];
    formatter.numberStyle = NSNumberFormatterCurrencyStyle;
    formatter.negativePrefix = [NSString stringWithFormat:@"-%@", formatter.currencySymbol];
    formatter.negativeSuffix = @"";
    return formatter;
}
@end
```

---

<br>

> #### 5. 十进制数字的四舍五入 {#p5}

```objc
@interface NSDecimalNumber (Helper)
+ (NSString *)stringOfCeilingDecimalNumber:(double)number WithScale:(NSInteger)scale;
+ (NSString *)currencyStringOfCeilingDecimalNumber:(double)number WithScale:(NSInteger)scale;
@end
```

```objc
@implementation NSDecimalNumber (Helper)
+ (NSDecimalNumber *)ceilingDecimalNumber:(double)number WithScale:(NSInteger)scale{
    NSDecimalNumberHandler* roundingBehavior = [NSDecimalNumberHandler decimalNumberHandlerWithRoundingMode:NSRoundUp scale:scale raiseOnExactness:NO raiseOnOverflow:NO raiseOnUnderflow:NO raiseOnDivideByZero:NO];
    NSDecimalNumber *decimalNumber = [[[NSDecimalNumber alloc] initWithString:[NSString stringWithFormat:@"%.6f", number]] decimalNumberByRoundingAccordingToBehavior:roundingBehavior];
    return decimalNumber;
}

+ (NSString *)stringOfCeilingDecimalNumber:(double)number WithScale:(NSInteger)scale{
    return [NSString stringWithFormat:@"%.2f", [self ceilingDecimalNumber:number WithScale:scale].doubleValue];
}

+ (NSString *)currencyStringOfCeilingDecimalNumber:(double)number WithScale:(NSInteger)scale{
    return [[NSNumberFormatter RMBFormatter] stringFromNumber:[self ceilingDecimalNumber:number WithScale:scale]];
}
@end
```

---

<br>

> #### 6. md5加密字符串 {#p6}

```objc
+ (NSString *)encodeStringUsingMd5:(NSString *)string{
    const char *cStr = [string UTF8String];
    unsigned char result[16];
    CC_MD5(cStr, (int)strlen(cStr), result); // This is the md5 call
    
    return [NSString stringWithFormat:
            @"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
            result[0], result[1], result[2], result[3],
            result[4], result[5], result[6], result[7],            
            result[8], result[9], result[10], result[11],
            result[12], result[13], result[14], result[15]
            ];
}
```

---

<br>

> #### 7. 16进制色值转颜色 {#p7}

```objc
+ (UIColor *)colorWithHexString: (NSString *)stringToConvert {
    NSString *cString = [[stringToConvert stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]] uppercaseString];
    
    //  string should be 6 or 8 characters
    if ([cString length] < 6) return [UIColor clearColor];
    
    //  strip 0X if it appears
    if ([cString hasPrefix:@"0X"]) cString = [cString substringFromIndex:2];
    if ([cString hasPrefix:@"#"]) cString = [cString substringFromIndex:1];
    if ([cString length] != 6) return [UIColor clearColor];

    //  separate into r, g, b substrings
    NSRange range;
    range.location = 0;
    range.length = 2;
    NSString *rString = [cString substringWithRange:range];
    
    range.location = 2;
    NSString *gString = [cString substringWithRange:range];
    
    range.location = 4;
    NSString *bString = [cString substringWithRange:range];
    
    //  scan values
    unsigned int r, g, b;
    [[NSScanner scannerWithString:rString] scanHexInt:&r];
    [[NSScanner scannerWithString:gString] scanHexInt:&g];
    [[NSScanner scannerWithString:bString] scanHexInt:&b];
    
    return [UIColor colorWithRed:((float) r / 255.0f)
                           green:((float) g / 255.0f)
                            blue:((float) b / 255.0f)
                           alpha:1.0f];
}
```

---

<br>

> #### 8. 颜色变暗 {#p8}

对 **UIColor** 类进行扩展

```objc
@implementation UIColor (DimmedColor)
- (UIColor *)dimmedColor: (CGFloat)dimmedDegree{
    CGFloat red;
    CGFloat green;
    CGFloat blue;
    CGFloat alpha;
    [self getRed:&red green:&green blue:&blue alpha:&alpha];
    red = MAX(red - dimmedDegree, 0.0);
    green = MAX(green - dimmedDegree, 0.0);
    blue = MAX(blue - dimmedDegree, 0.0);
    alpha = 1.0;
    return [UIColor colorWithRed:red green:green blue:blue alpha:alpha];
}
@end

```
