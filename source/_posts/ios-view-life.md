title: "ViewController生命周期控制方法"
date: 2015-04-22 18:12:44
tags: "lifecycle"
categories: ["iOS","Objective-C"]
---

> [视图控制对象生命周期-init、viewDidLoad、viewWillAppear、viewDidAppear、viewWillDisappear等的区别及用途 ](http://blog.csdn.net/weasleyqi/article/details/8090373)
> [详细了解 viewcontroller的生命周期](http://blog.csdn.net/fanjunxi1990/article/details/8699874)

iOS视图控制对象生命周期-init、viewDidLoad、viewWillAppear、viewDidAppear、viewWillDisappear、viewDidDisappear的区别及用途

init－初始化程序
viewDidLoad－加载视图
viewWillAppear－UIViewController对象的视图即将加入窗口时调用；
viewDidApper－UIViewController对象的视图已经加入到窗口时调用；
viewWillDisappear－UIViewController对象的视图即将消失、被覆盖或是隐藏时调用；
viewDidDisappear－UIViewController对象的视图已经消失、被覆盖或是隐藏时调用；
viewVillUnload－当内存过低时，需要释放一些不需要使用的视图时，即将释放时调用；
viewDidUnload－当内存过低，释放一些不需要的视图时调用。

视图控制对象通过alloc和init来创建，但是视图控制对象不会在创建的那一刻就马上创建相应的视图，而是等到需要使用的时候才通过调用loadView来创建，这样的做法能提高内存的使用率。比如，当某个标签有很多UIViewController对象，那么对于任何一个UIViewController对象的视图，只有相应的标签被选中时才会被创建出来。
