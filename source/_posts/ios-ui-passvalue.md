title: "iOS 页面之间的传值"
date: 2015-04-10 15:29:57
tags:
categories: ["iOS","Objective-C"]
---

> [【IOS开发】 UIView之间常用四种传值方式](http://blog.csdn.net/penuel/article/details/8857947)

###### 直接传值

从父页面传递到子页面，或者从上一级页面传递到下一级页面，在执行`presentViewController:animated:completion:`或者`pushViewController:animated:`方法转移页面。
在子类定义一个属性
```objc
@property(strong,nonatomic) NSString *msg;
```

在父类直接赋值即可
```objc
SecondViewController *nextController = [[SecondViewController alloc] init];
nextController.msg = @"Next";
[self presentViewController:nextController animated:YES completion:^{}];
```

###### 协议传值

从子页面传递到父页面，或者从下一级页面传递到上一级页面，在执行`dismissViewControllerAnimated:completion:`或者`popView`方法转移页面。
首先建立一个协议
```objc
@protocol passValueDelegate <NSObject>

-(void)passValue:(NSString *)str;

@end
```

在子页面添加协议属性
```objc
@property(nonatomic,weak) id<passValueDelegate> delegate;
```

在子页面中调用
```objc
[self.delegate passValue:@"back"];
```

在父页面中实现该协议，并且定义自己为协议
```objc
@interface ViewController ()<passValueDelegate>
-(void)passValue:(NSString *)str
{
    NSLog(@"%@",str);
}

SecondViewController *nextController = [[SecondViewController alloc] init];
nextController.msg = @"Next";
//在调用子页面之前，需要指定代理对象为自身
nextController.delegate = self;
[self presentViewController:nextController animated:YES completion:^{
    
}];
```

###### NSUserDefaults传值
这种方式辉在设备上留下数据，需要谨慎使用
```objc
//写入值
[[NSUserDefaults standardUserDefaults] setObject:@"test" forKey:@"third"];
[[NSUserDefaults standardUserDefaults] synchronize];

//在其他页面读取
[[NSUserDefaults standardUserDefaults] objectForKey:@"third"];
```
