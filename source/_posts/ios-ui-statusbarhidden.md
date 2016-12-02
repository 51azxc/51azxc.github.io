title: "iOS 隐藏状态栏"
date: 2015-12-23 18:24:58
tags:
categories: ["iOS"]
---

> [IOS7如何隐藏状态栏，貌似之前的没效果了](http://www.cocoachina.com/bbs/read.php?tid-153922.html)
> [How do I hide the status bar in a Swift iOS app?](http://stackoverflow.com/questions/24236912/how-do-i-hide-the-status-bar-in-a-swift-ios-app)

在需要隐藏状态栏的`ViewController`中重写`prefersStatusBarHidden`方法
```objc
- (BOOL)prefersStatusBarHidden {
    return YES;
}
```
然后在需要更改隐藏状态的地方调用`setNeedsStatusBarAppearanceUpdate`方法
```objc
[self setNeedsStatusBarAppearanceUpdate]
```

`swift`写法
```swift
override func prefersStatusBarHidden() -> Bool {
    return true
}
```