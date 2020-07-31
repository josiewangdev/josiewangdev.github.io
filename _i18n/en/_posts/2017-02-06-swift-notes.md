---
layout: post
title: Swift Notes
tags: Notes
categories: Dev
---

## Swift Notes

---

#### 1. [Access control](#access-control)
#### 2. [Associate value in enum](#enum-associate)
#### 3. [Encode enum type](#enum-encode)
#### 4. [Identity operator](#identity-operator)
#### 5. [Higher order fuctions](#higher-order-funcs)
#### 6. [String to decimal number](#string-to-number)
#### 7. [Chinese string to Pinyin](#string-to-pinyin)
#### 8. [Generics](#generics)
#### 9. [Naming conflict](#naming-conflict)
#### 10. [Sigleton](#singleton) 
#### 11. [Init array using repeated value](#repeated-value-array)
#### 12. [Transform between class name and class](#class-and-name)
#### 13. [Predicate Queries](#predicate)
#### 14. [Break out of multiple loop levels using statements](#break-multiple-loop)
#### 15. [Subscript](#subscript)

---

<br>

> #### 1. Access control {#access-control}

In Swift, access control restricts access to parts of your code from code in other source files and modules.

This feature enables you to hide the implementation details of your code, and to specify a preferred interface through which your code could be access and used.

Access levels shown like following: 

**`open` > `public` > `internal`(defualt) > `fileprivate` > `private`**


`open` access and `public` access enable entities to be used within any source file from their definine module, and also in a source file from another module that imports the defining module. You typically use `open` or `public` accesss when specifying the public interface to a framework.

`open` access differs from `public` access by allowing code outside the module to subclass and override, but `public` only enables subclass and override in the module.

`internal` access enables entities to be used within any source file from their defining module, but not in any source file outside of that module. You typically use `internal` access when defining an app’s or a framework’s internal structure.

`fileprivate` access restricts the use of an entity to its own defining source file.

`private` access restricts the use of an entity to the enclosing declaration, and to extensions of that declaration that are in the same file. 

---

<br>

> #### 2. Associate value in enum {#enum-associate}

In Swift, enum cases could carry many types of values.

```swift
enum Value {
    case valueInt(Int)
    case valueFloat(Float)
    case valueString(String)
}

func describe(value: Value) -> String {
    switch value {
    case .valueInt(let int):
        return "Result: \(int)"
    case .valueFloat(let float):
        return "Result: \(float)"
    case .valueString(let string):
        return "Result: \(string)"
    }
}

describe(Value.valueInt(1234))
//return: "Result: 1234"
describe(Value.valueFloat(3.14))         
//return: "Result: 3.14"
describe(Value.valueString("hello"))     
//return: "Result: hello"
```

---

<br>

> #### 3. Encode enum type {#enum-encode}

If we want to make the Enum type encodable, we need to convert the enum to and from the raw value like this.

```swift

enum Stage: String {
    case DisplayAll = "displayAll"
}

class AppState : NSObject, NSCoding {
    var idx   = 0
    var stage = Stage.DisplayAll

    override init() {}

    required init(coder aDecoder: NSCoder) {
        self.idx = aDecoder.decodeIntegerForKey("idx")
        self.stage = Stage(rawValue: (aDecoder.decodeObjectForKey("stage") as! String)) ?? .DisplayAll
    }

    func encodeWithCoder(aCoder: NSCoder) {
        aCoder.encodeInteger(self.idx, forKey: "idx")
        aCoder.encodeObject(self.stage.rawValue, forKey:"stage")
    }
}
```

---

<br>

> #### 4. Identity operator {#identity-operator}

`==` and `!=` are operators to check if their instance values are equal, means *equal to* .

`===` and `!==` are identity operators and used to check if the references point the same instance, means *identical to* .

```swift
if anyObject1 == anyObject2 {
    //to check whether their values are equal
}

if anyObject1 === anyObject2 {
    //to check whether they are the same instance
}
```

---

> #### 5. Higher order fuctions {#higher-order-funcs}

**Reduce**

Use reduce to combine all items in a collection to create a single new value.

```swift
let items = [2.0,4.0,5.0,7.0]
let total = items.reduce(10.0,combine: +)
//  28.0

let items = [2.0,4.0,5.0,7.0]
let total = items.reduce(10.0,+)
//  28.0

let codes = ["abc","def","ghi"]
let text = codes.reduce("", combine: +)
//  "abcdefghi"

let codes = ["abc","def","ghi"]
let text = codes.reduce("",+)
//  "abcdefghi"
```

**FlatMap**

Flatmap is used to flatten a collection of collections.

```swift
let collections = [[5,2,7],[4,8],[9,1,3]]
let flat = collections.flatMap { $0 }
//  [5, 2, 7, 4, 8, 9, 1, 3]
```

Flatmap is also used for removing nil values from a collection. 

```swift
let people: [String?] = ["Tom",nil,"Peter",nil,"Harry"]
let valid = people.flatMap {$0}
//  ["Tom", "Peter", "Harry"]
```

---

<br>

>#### 6. String to decimal number {#string-to-number}

**Method 1**

```swift
let str : NSString = oriString.text
var floatVal : Float = str.floatValue
var doubleVal : Double = str.doubleValue
```

**Method 2**

```swift
let formatter = NSNumberFormatter()
formatter.numberStyle = NSNumberFormatter.DecimalStyle
var number : NSNumber? = formatter.numberFromString(oriString)
var floatVal = number.floatValue
var doubleVal = number.doubleValue
```

---

<br>

> #### 7. Chinese string to Pinyin {#string-to-pinyin}

```swift
/// - Parameter hasBlank: contains blank or not
func transformToPinyin(hasBlank: Bool =false) -> String {
    let stringRef = NSMutableString(string: self) as CFMutableString
    CFStringTransform(stringRef,nil, kCFStringTransformToLatin, false) // 转换为带音标的拼音
    CFStringTransform(stringRef, nil, kCFStringTransformStripCombiningMarks, false) // 去掉音标
    let pinyin = stringRef as String
    return hasBlank ? pinyin : pinyin.replacingOccurrences(of: " ", with: "")
}
```

**Convert to string with Pinyin initials**

```swift
/// - Parameter lowercased: 是否小写（默认大写）
func transformToPinyinHead(lowercased: Bool =false) -> String {
    let pinyin = self.transformToPinyin(hasBlank: true).capitalized // 字符串转换为首字母大写
    var headPinyinStr = ""
    for ch in pinyin.characters {
        if ch <= "Z" && ch >= "A" {
            headPinyinStr.append(ch) // 获取所有大写字母
        }
    }
    return lowercased ? headPinyinStr.lowercased() : headPinyinStr
}
```

---

<br>

> #### 8. Generics {#generics}

Generic code enables you to write flexible, reusable functions and types that can work with any type, subject to requirements that you define. You can write code that avoids duplication and expresses its intent in a clear, abstracted manner.

As example, following is getting a perferred type of the parent views:

```swift
extension UIView {
    func superView<T: UIView>(of: T.Type) -> T? {
        for view in sequence(first: self.superview, next: { $0?.superview }) {
            if let father = view as? T {
                return father
            }
        }
        return nil
    }
}
```

```swift
// get the parent view which is a UITableView
let superView = view.superView(of: UITableView.self)
```

---

<br>

#### 9. Naming conflict {#naming-conflict}

A Swift framework called **FrameworkA** that defines the type **Thing**.

A Swift framework called **FrameworkB** that also defines the type **Thing** and the type **FrameworkA**.

An app that imports both frameworks in the same Swift file.

**Q:** How to reference **FrameworkA.Thing** in said file?

**A:** Use **`typealias`** to redefine **Thing** to solve the problem.

```swift
import FrameworkA
typealias ThingA = Thing
let t = FrameworkA.ThingA
```

---

<br>

> #### 10. Sigleton {#singleton}

In Swift, the lazy initializer for a global variable (also for static members of structs and enums) is run the first time that global is accessed, and is launched as `dispatch_once` to make sure that the initialization is atomic. This enables a cool way to use `dispatch_once` in your code: just declare a global variable with an initializer and mark it private.

Also, we all know that the static variable still support the `dispatch_once` feature, Swift will implement `swift_once_block_invoke` when it is inited, so we can use a simpler way to implement singleton.

```swift
//use global variable to implement singleton
private let sharedClassName = TheOneAndOnlyClass()
class TheOneAndOnlyClass {
    class var sharedInstance: TheOneAndOnlyClass {
        return sharedClassName
    }
}
```

```swift
//a more simpler way to implement singleton
class TheOneAndOnlyClass {
    static let sharedInstance = TheOneAndOnlyClass()
    private init() {} //This prevents others from using the default '()' initializer for this class.
}
```

---

<br>

> #### 11. Init array using repeated value {#repeated-value-array}

When you use the function `init(count:, repeatedValue:)` to initialize an array, you should know each value of this array references the same instance. So if you change anyone of them, all the others ones would also be changed.

```swift
let array = [AnyObject](count: count, repeatedValue: AnyObject().init())
```

---

<br>

> #### 12. Transform between class name and class {#class-and-name}

**Get class from class name string**

We can use `NSClassFromString` to get the instance of a class from class name by using

```swift
let model = NSClassFromString("bundleName.className") as! NSObject.Type;
let enity = model.init();
```

But in Swift, the `className` needs to be appended by app's name as prefix, usually app name looks like this: **myAppName_switch**, so we need to change code like this

```swift
if let appName: String = NSBundle.mainBundle().objectForInfoDictionaryKey("CFBundleName") as! String? {

    //get the appName
    let indexStart = appName.startIndex.advancedBy(10)
    let indexEnd = appName.endIndex.advancedBy(-6)
    let subStr = appName.substringToIndex(indexStart)
    let subStr2 = appName.substringFromIndex(indexEnd)
    let endName = "\(subStr)_\(subStr2)"

    //get the className string as appName + className
    let classStringName = "\(endName).\(className)"

    //get the instance
    let model = NSClassFromString(classStringName)
    let entity = myClass.init()

}
```
或者可以这样说， NSClassFromString 在Swift中的参数不只是一个单独的类字符串,而是一个完整的包名加类名组成的字符串,也就是包类名字符串.

如下：

```swift
let model = NSClassFromString("包名.类名") as! NSObject.Type;
let enity = model.init();
```

**Get class name string from class**

```swift
//  from a class
String(describing: YourClass.self)
//  from a type
String(describing: self)
```

```swift
//Example
struct Foo {
    // Instance Level
    var typeName: String {
        return String(describing: Foo.self)
    }
    // Instance Level - Alternative Way
    var otherTypeName: String {
        let thisType = type(of: self)
        return String(describing: thisType)
    }
    // Type Level
    static var typeName: String {
        return String(describing: self)
    }
}

Foo().typeName       // = "Foo"
Foo().otherTypeName  // = "Foo"
Foo.typeName         // = "Foo"
```

---

<br>

> #### 13. Predicate Queries {#predicate}

Using `NSPredicate` in Swift

```swift

let predicate = NSPredicate(format: "name = %@", txtFieldName.text)
let predicate = NSPredicate(format: "name = %@ AND nickName = %@", argumentArray: [name, nickname])
let predicate = NSPredicate(format: "name contains[c] %@", searchText)
let predicate = NSPredicate(format: "name contains[c]\(searchText)", nil)
let predicate = NSPredicate(format: "name contains[c] %@ AND nickName contains[c] %@", argumentArray: [name, nickname])
let predicate = NSPredicate(format: "SELF contains %@", searchText)

let searchResults = dataSource.filter { predicate.evaluateWithObject($0) }
var searchResults = dataSource.filteredArrayUsingPredicate(predicate)

```

---

<br>

> #### 14. Break out of multiple loop levels using statements {#break-multiple-loop}

In Swift’s labeled statements, execution pickes up directly after the block we labeled. 

So if we're using two loops or more and want to break out ot them all, labeled statements could solve this problem.

```swift
loopOne: for item in itemGroup {
    for subItem in item.subItems {
        if sampleId == subItem.id {
            ...
            break loopOne
        }
    }
}
```

---

<br>

> #### 15. 下标subscript {#subscript}

下标 `subscript` 在数组和字典中使用，但是你可以给任何类型（枚举，结构体，类）增加 下标 `subscript` 的功能。

`subscript` 的语法类似实例方法、计算属性，其本质就是方法。例如给 **Matix** 类增加 **indexPath** 下标功能获取其中 **data** 的值。

```swift
class Matix {
    var data = [
        [1,0,0],
        [0,1,0],
        [0,0,1]
    ]
    subscript(indexPath: IndexPath) -> Int {
        set {
            guard indexPath.section >= 0 && indexPath.section < 3 && indexPath.row >= 0 && indexPath.row < 3 else {
                return
            }
            data[indexPath.section][indexPath.row] = newValue
        }
        get {
            guard indexPath.section >= 0 && indexPath.section < 3 && indexPath.row >= 0 && indexPath.row < 3 else {
                return 0
            }
            return data[indexPath.row][indexPath.section]
        }
    }
}

var m = Matix()
m[IndexPath(row: 1, section: 1)] = 3
print(m.data)  //  [[1, 0, 0], [0, 3, 0], [0, 0, 1]]
print(m[IndexPath(row: 0, section: 0)]) //  1
```
---




