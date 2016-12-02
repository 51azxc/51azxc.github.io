title: "iOS延时操作"
date: 2015-05-12 17:45:24
tags: ["timer","thread","performSelector"]
categories: ["iOS","Objective-C"]
---

### Ojbective-C中延时操作
> [How to Wait in Objective C](http://stackoverflow.com/questions/6983400/how-to-wait-in-objective-c)

1.使用`performSelector`方法,指定`afterDelay`为延时长度
```objc
[self performSelector:@selector(callMethod:) withObject:nil afterDelay:0.5];
```

2.在方法中使用`NSThread`的`sleepForTimeInterval`也可进行短暂挂起。
```objc
[NSThread sleepForTimeInterval:0.5f];
```

3.使用`NSTimer`定时器延时触发任务
```objc
[NSTimer scheduledTimerWithTimeInterval:0.5
         target:self
         selector:@selector(callMethod:)
         userInfo:nil
         repeats:NO];


//需要调用的方法
- (void) callMethod:(NSTimer*)t {

}
```
