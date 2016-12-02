title: "iOS 旋转屏幕"
date: 2015-04-13 14:13:52
tags: "rorate"
categores: ["iOS","Objective-C"]
---

> [iOS6的旋屏控制技巧](http://blog.csdn.net/yiyaaixuexi/article/details/8035014)
> [问题解决:iOS6下shouldAutorotateToInterfaceOrientation不起作用，屏幕旋转同时支持iOS5和iOS6](http://blog.163.com/l1_jun/blog/static/14386388201302294653607/)

```objc
//只支持横屏
- (NSUInteger)supportedInterfaceOrientations
{
    return UIInterfaceOrientationMaskLandscape;
}
//开启旋转
- (BOOL)shouldAutorotate
{
    return YES;
}
//初始屏幕旋转方向 
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
    return UIInterfaceOrientationPortrait;
}
```
