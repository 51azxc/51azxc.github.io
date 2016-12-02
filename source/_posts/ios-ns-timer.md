title: "NSTimer的使用"
date: 2015-04-02 16:05:14
tags: ["timer", "GCD"]
categories: ["iOS","Objective-C"]
---

### 基本使用

> [NSTimer类的使用](http://www.cnblogs.com/wujian1360/archive/2011/09/05/2167992.html)
> [Passing parameters to the method called by a NSTimer](http://stackoverflow.com/questions/4011297/passing-parameters-to-the-method-called-by-a-nstimer)

```objc
NSTimer timer1 = [NSTimer scheduledTimerWithTimeInterval:34 
                          target:self
                          selector:@selector(uploadLog:)
                          userInfo:@"count" repeats:YES];
- (void)uploadLog: (NSTimer *)timer {
    NSLog(@"count: %@", (NSString *)[timer userInfo]);
} 
```
以上代码可以新建一个定时器，第一个参数为时间间隔数，单位为秒；target表示发送对象；selector表示制定时刻调用什么方法；userInfo可以给指定方法传递参数，无需可指定为`nil`；repeats表示是否重复执行定时器。
使定时器失效可以调用`[timer1 invalidate]`方法。

----

### GCD中使用
> 来自[Timer inside global queue is not calling in iOS](http://stackoverflow.com/questions/14569693/timer-inside-global-queue-is-not-calling-in-ios)

如果使用了GCD，则需要在主线程切换回来才能实现效果，如
```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	NSTimer *timer = [NSTimer timerWithTimeInterval:0.10 
									 target:self 
								   selector:@selector(action_Timer) 
								   userInfo:nil 
									repeats:YES];
	dispatch_async(dispatch_get_main_queue(), ^{
		[[NSRunLoop mainRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
	});        
});
```
