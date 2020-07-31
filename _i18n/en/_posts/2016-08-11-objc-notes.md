---
layout: post
title: Objective-C Notes
tags: Notes
categories: Dev
---

## Objective-C Notes

---

#### 1. [Singleton](#singleton)
#### 2. [Keyword __block](#block)
#### 3. [Predicate Queries](#predicate)

---

<br>

> #### 1. Singleton {#singleton}

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

> #### 2. Keyword __block {#block}

Normally, variables that are used in blocks are copied, thus any modification done to these variables don't show outside the block. 

![block](/images/block/block2.png)

```objc
char localCharacter;

void (^aBlock)(void) = ^(void) {
    int c;
    c = localCharacter; // use localCharacter value but cannot change it
};
```

When they are marked with `__block` , the modifications done inside the block are also visible outside of it, because the variables are no longer copied, but directly used as the non-local variables.

![block](/images/block/block1.png)

```objc
__block char localCharacter;

void (^aBlock)(void) = ^(void) {
    localCharacter = 'a'; // could change localCharacter value in enclosing scope
};
```

---

<br>

> #### 3. Predicate Queries {#predicate}

Using `NSPredicate` in Objective-C

```objc
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name contains[c] %@", searchText];
```



