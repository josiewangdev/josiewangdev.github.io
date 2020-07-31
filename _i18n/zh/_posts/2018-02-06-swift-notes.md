---
layout: post
title: Swift 随笔
tags: Notes
categories: Dev
---

## Swift 随笔

---

#### 1. [访问控制](#access-control)
#### 2. [枚举的关联值](#enum-associate)
#### 3. [枚举的编码](#enum-encode)
#### 4. [恒等运算符](#identity-operator)
#### 5. [高阶函数](#higher-order-funcs)
#### 6. [字符串转换为数字](#string-to-number)
#### 7. [将中文字符串转换为拼音](#string-to-pinyin)
#### 8. [泛型](#generics)
#### 9. [命名冲突](#naming-conflict)
#### 10. [单例](#singleton)
#### 11. [用repeated value初始化的浅拷贝数组](#repeated-value-array)
#### 12. [类名与类的转换](#class-and-name)
#### 13. [Predicate 查询语句](#predicate)
#### 14. [跳出多层循环](#break-multiple-loop)
#### 15. [下标subscript](#subscript)

---

<br>

> #### 1. 访问控制 {#access-control}

Swift 中, 访问控制限制了从其他源文件或者模块访问你的某部分代码的权限。

这个特性可以让你将实现代码的细节对外隐藏，并且能通过指定接口暴露可以被访问和使用的部分代码。

访问级别如下

**`open` > `public` > `internal`(默认) > `fileprivate` > `private`**

`open` 和 `public` 修饰的实体可以在其被定义的模块中的任何地方访问，也可以在导入其定义模块后的其他模块中访问。 通常情况下，你可以使用 `open` 或者 `public` 去声明一个框架的开放接口。 

`open` 和 `public` 的不同之处在于 `open` 可以让在其定义模块外的代码中继承或重写。而  `public` 只可在本模块内重写。

`internal` 修饰的实体可以在本模块内的任意其他文件中访问，对自身模块开放所有源文件，对其他所有外界源代码屏蔽。 通常可以使用 `internal` 定义应用内或者框架内的内部结构。

`fileprivate` 修饰的属性只能在其定义的当前文件内访问。

`private` 只能在其声明的类（作用域）内访问，以及当前文件内的声明类的扩展类中访问。

---

<br>

> #### 2. 枚举的关联值 {#enum-associate}

在Swift中，枚举可以携带不同的值。

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

> #### 3. 枚举的编码 {#enum-encode}

如果想要编码枚举类型，我们需要将枚举类型的原始值进行转换。

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

> #### 4. [恒等运算符](#identity-operator)

`==` 和 `!=` 运算符用来判断两个对象的值是否相等。

`===` 和 `!==` 是恒等运算符，是用来判断两个对象是否引用了同一个实例，即用来判断这两个是不是同一个对象。

```swift
if anyObject1 == anyObject2 {
    //  check whether their values are equal
}

if anyObject1 === anyObject2 {
    //  check whether they are the same instance
}
```

---

<br>

> #### 5. 高阶函数 {#higher-order-funcs}

**Reduce**

使用reduce来组合集合中的所有元素并返回一个新的非集合类型的值。

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

Flatmap用于将一个二维数组拆开展平.

```swift
let collections = [[5,2,7],[4,8],[9,1,3]]
let flat = collections.flatMap { $0 }
//  [5, 2, 7, 4, 8, 9, 1, 3]
```

Flatmap 也可以用来判断集合中的空值，并将空值移出集合。

```swift
let people: [String?] = ["Tom",nil,"Peter",nil,"Harry"]
let valid = people.flatMap {$0}
//  ["Tom", "Peter", "Harry"]
```

---

<br>

> #### 6. 字符串转换为数字 {#string-to-number}

**方法1**

```swift
let str : NSString = oriString.text
var floatVal : Float = str.floatValue
var doubleVal : Double = str.doubleValue
```

**方法2**

```swift
let formatter = NSNumberFormatter()
formatter.numberStyle = NSNumberFormatter.DecimalStyle
var number : NSNumber? = formatter.numberFromString(oriString)
var floatVal = number.floatValue
var doubleVal = number.doubleValue
```

---

<br>

> #### 7. 将中文字符串转换为拼音 {#string-to-pinyin}

```swift
/// - Parameter hasBlank: 是否带空格（默认不带空格）
func transformToPinyin(hasBlank: Bool =false) -> String {
    let stringRef = NSMutableString(string: self) as CFMutableString
    CFStringTransform(stringRef,nil, kCFStringTransformToLatin, false) // 转换为带音标的拼音
    CFStringTransform(stringRef, nil, kCFStringTransformStripCombiningMarks, false) // 去掉音标
    let pinyin = stringRef as String
    return hasBlank ? pinyin : pinyin.replacingOccurrences(of: " ", with: "")
}
```

**转换为中文拼音首字母字符串**

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

> #### 8. 泛型 {#generics}

泛型可以让你的代码更灵活且复用性高。泛型可以作用于任意符合你定义的类型的对象。使用泛型可以避免代码的重复并且可以将你的代码浓缩至干净的抽象的结构。

以获取某一特定类型的父级视图代码举例:

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
//  get the parent view which is a UITableView
let superView = view.superView(of: UITableView.self)
```

---

<br>

> #### 9. 命名冲突 {#naming-conflict}

**FrameworkA** 定义了类型 **Thing**。

**FrameworkB** 也定义了类型 **Thing** 以及使用了 **FrameworkA**。

在一个App的Swift文件中同时导入了 **FrameworkA** 和 **FrameworkB**。

**Q:** 怎样才能在这个文件中引用 **FrameworkA.Thing**?

**A:** 使用 **`typealias`** 为 **Thing** 定义别名解决问题。

```swift
import FrameworkA
typealias ThingA = Thing
let t = FrameworkA.ThingA
```

---

<br>

> #### 10. 单例 {#singleton}

在Swift中，全局变量（还有结构体和枚举体的静态成员）的懒加载初始化方法会在其被访问的时候调用一次，类似于调用 `dispatch_once` 以保证其初始化的原子性。这样就有了一种很酷的 `dispatch_once` 方式：只声明一个全局变量和私有的初始化方法即可。

我们又已知静态类变量是支持 `dispatch_once` 特性的, Swift将会把静态变量的初始化包装在一次 `swift_once_block_invoke` 中，所以就有了一种更加简洁直观的单例写法。

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

> #### 11. 用repeated value初始化的浅拷贝数组 {#repeated-value-array}
如果你使用了 `init(count:, repeatedValue:)` 方法进行数组初始化，则你需要知道这个数组是创建了多个指向同一个内存地址的指针，所以当你改变了数组中任意的一个值时，其他所有的值都会被改变。

```swift
let array = [AnyObject](count: count, repeatedValue: AnyObject().init())
```

---

<br>

> #### 12. 类名与类的转换 {#class-and-name}

**从类名获取类**

我们可以用 `NSClassFromString` 从类名来获取到一个该类的实例。

```swift
let model = NSClassFromString("bundleName.className") as! NSObject.Type;
let enity = model.init();
```

但是在Swift中，`className`是由类名再加上项目应用名称作为前缀组成, 并且swift中的应用命名方式是这样的：**myAppName_switch**, 所以我们需要将代码改为

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

**从类获取类名**

```swift
//  from a class
String(describing: YourClass.self)
//  from a type
String(describing: self)
```

```swift
//  Example
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

> #### 13. Predicate 查询语句 {#predicate}

Swift中使用 `NSPredicate` 的方法

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

> #### 14. 跳出多层循环 {#break-multiple-loop}

使用Swift的标签语句时，执行语句会直接选择我们命名的block。

所以当我们使用了多层循环，并且在某个时刻想要跳出全部循环（或者也可以理解为跳出指定循环），则可以像这样使用标签语句来实现。

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

