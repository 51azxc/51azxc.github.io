title: "iOS 动画"
date: 2015-04-21 17:35:23
tags: "animation"
categories: ["iOS","Objective-C"]
---

### IOS Core Animation

> [<iOS> 谈谈iOS Animation](http://blog.csdn.net/smking/article/details/8424851)
> [<原>关键帧动画CAKeyframeAnimation](http://blog.csdn.net/huifeidexin_1/article/details/8504075)
> [IOS动画Core Animation详解](http://blog.csdn.net/wildfireli/article/details/23191693)
> [ IOS中通过Core Animation实现简单动画](http://blog.csdn.net/kut00/article/details/8141440)
> [_OBJC_CLASS_$_CALayer"问题解](http://yul100887.blog.163.com/blog/static/200336135201272435324368/)

* 通过动画上下文使用UIKit动画
```objc
UIView *view1 = [[UIView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
view1.backgroundColor = [UIColor blueColor];
[self.view addSubview:view1];
//开始动画
[UIView beginAnimations:@"view1" context:nil];
//动画时长
[UIView setAnimationDuration:5];
//开始动画设置

view1.backgroundColor = [UIColor purpleColor];
view1.frame = CGRectMake(50, 50, 50, 50);
view1.backgroundColor = [UIColor colorWithRed:100/255.0 green:222/255.0 blue:123/255.0 alpha:0.5];

//动画结束
[UIView commitAnimations];
```

* 通过`animateWithDuration`方法实现动画
```objc
[UIView animateWithDuration:5 animations:^{
    view1.backgroundColor = [UIColor purpleColor];
    view1.frame = CGRectMake(50, 50, 50, 50);
    view1.backgroundColor = [UIColor colorWithRed:100/255.0 green:222/255.0 blue:123/255.0 alpha:0.5];
} completion:^(BOOL finished) {
    NSLog(@"finish");
}];
```

* 使用Core Animation对象来实现动画
通过`CAKeyframeAnimation`实现移动动画
```objc
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
//初始化路径
CGMutablePathRef path1 = CGPathCreateMutable();
//动画起点
CGPathMoveToPoint(path1, nil, 50, 60);
CGPathAddCurveToPoint(path1, nil, 111, 22, 333, 44, 55, 66);
[animation setPath:path1];
[animation setDuration:6];
//动画回到原位
[animation setAutoreverses:YES];
//设置渐出效果
animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
//自动旋转
animation.rotationMode = @"auto";

[view1.layer addAnimation:animation forKey:@"position"];
```
通过`CABasicAnimation`实现形变动画
```
[CATransaction begin];
[CATransaction setValue:[NSNumber numberWithInt:6] forKey:kCATransactionAnimationDuration];
//按x轴旋转
CABasicAnimation *flipAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.x"];
flipAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
//旋转180度,即M_PI
flipAnimation.toValue = [NSNumber numberWithFloat:M_PI];
[flipAnimation setDuration:6];
//旋转后保持状态
[flipAnimation setFillMode:kCAFillModeForwards];
[flipAnimation setRemovedOnCompletion:NO];
[view1.layer addAnimation:flipAnimation forKey:@"flip"];
[CATransaction commit];
```

----

出现**_OBJC_CLASS_$_CALayer**错误是需要在项目*Framework*中添加`QuartzCore.framework`

----

### 自定义PresentViewController/Push/Pop动画

> [How to change the Push and Pop animations in a navigation based app](http://stackoverflow.com/questions/2215672/how-to-change-the-push-and-pop-animations-in-a-navigation-based-app)
> [How to custom Modal View Controller presenting animation?](http://stackoverflow.com/questions/19931710/how-to-custom-modal-view-controller-presenting-animation)

使用`CAtransition`类来更改过场动画

```objc
CATransition *transition = [CATransition animation];
[transition setDuration:1];
[transition setType:kCATransitionMoveIn];
//从右边载入
[transition setSubtype:kCATransitionFromRight];
[self.view.window.layer addAnimation:transition forKey:kCATransition];
AlertViewController *alertViewController = [[AlertViewController alloc] init];
[self presentViewController:alertViewController animated:NO completion:nil];
```