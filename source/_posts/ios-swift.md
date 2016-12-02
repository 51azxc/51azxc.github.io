title: Swift实用功能
date: 2015-10-19 19:04:26
tags: ["date", "color"]
categories: ["iOS","swift"]
---

### `nil`聚合运算

> [Swift学习笔记（十二）——nil的聚合运算 Nil Coalescing Operator](http://blog.csdn.net/chenyufeng1991/article/details/47072623)

初次见到`??`符号很是奇特，赶紧查了下才发现原来是三目运算符的精简模式，等同于`a != nil ? a! : b`;因为**Swift**中的可选类型，因此需要判断其是否不为`nil`，使用这个运算符可以极大的简化代码
```swift
var a:String?
a = "a"
var b:String = a ?? "b"
print(b)
```

----

### 给Date添加输出当前日期功能

> [NSDateFormatter.stringFromDate(NSDate()) returns empty string](http://stackoverflow.com/questions/28332946/nsdateformatter-stringfromdatensdate-returns-empty-string)

```swift
extension DateFormatter {
    convenience init(dateStyle: DateFormatter.Style) {
        self.init()
        self.dateStyle = dateStyle
    }
}

extension Date {
    struct Formatter {
        static let shortDate = DateFormatter(dateStyle: .short)
    }
    var shortDate: String {
        return Formatter.shortDate.string(from: self)
    }
}

print(Date().shortDate)
```

----

### 随机生成颜色

> [How to make a random background color with Swift](http://stackoverflow.com/questions/29779128/how-to-make-a-random-background-color-with-swift)

```swift
extension CGFloat {
    static func random() -> CGFloat {
        return CGFloat(arc4random()) / CGFloat(UInt32.max)
    }
}

extension UIColor {
    static func randomColor() -> UIColor {
        return UIColor(red:   .random(),
                       green: .random(),
                       blue:  .random(),
                       alpha: 1.0)
    }
}

self.view.backgroundColor = .randomColor()
```
