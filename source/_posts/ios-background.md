title: "当应用切换到后台时调用方法"
date: 2015-04-03 16:43:35
tags:
categories: ["iOS","Objective-C"]
---

> [Use of background/foreground methods in AppDelegate](http://stackoverflow.com/questions/4846822/use-of-background-foreground-methods-in-appdelegate)

当app切换到后台后会触发`AppDelegate`中的`applicationDidEnterBackground`方法，其他对象就会收到`UIApplicationDidEnterBackgroundNotification`通知，因此可以在接收到这个通知的时候调用相应的方法
```objc
[[NSNotificationCenter defaultCenter] addObserver:self
                                      selector:@selector(appHasGoneInBackground:)
                                      name:UIApplicationDidEnterBackgroundNotification
                                      object:nil];
```

需要注意的是不需要监听通知的时候需要把他注销
```objc
[[NSNotificationCenter defaultCenter] removeObserver:self];
```
以下是`AppDelegate`中各种状态变化方法所发送的通知：
- `UIApplicationDidEnterBackgroundNotification`
- `UIApplicationWillEnterForegroundNotification`
- `UIApplicationWillResignActiveNotification`
- `UIApplicationDidBecomeActiveNotification`
