title: "NSDate的使用"
date: 2015-04-02 17:58:33
tags: "date"
categories: ["iOS","Objective-C"]
---

> 来自
> [NSDate 的一些操作](http://blog.csdn.net/reylen/article/details/8560128)
> [Objective-C学习之NSDate简单使用说明](http://mobile.51cto.com/iphone-407794.htm)
> [NSTimeInterval to HH:mm:ss?](http://stackoverflow.com/questions/4933075/nstimeinterval-to-hhmmss)

### 初始化日期
```objc
NSDate *date = [NSDate date];
```
时区转换
```objc
NSTimeZone *zone = [NSTimeZone systemTimeZone];
NSInteger interval = [zone secondsFromGMTForDate:date];
NSDate *localDate = [date dateByAddingTimeInterval:interval];
```

----

### 格式化时间
```objc
NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
//大写HH为24小时制,小写hh为12小时制
[formatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
NSString *dateStr = [formatter stringFromDate:date];
//string -> date
NSDate *strDate = [formatter dateFromString:dateStr];
//NSTimeInterval -> NSInteger
NSTimeInterval interval = [strDate timeIntervalSince1970];
//四舍五入
NSInteger i = round(interval);
```
根据当前时间新增时间，间隔单位为秒，负数则为之前的日期
```objc
NSDate *date1 = [[NSDate alloc] initWithTimeInterval:60 sinceDate:date];
//获取其中更早的日期
NSDate *eDate = [date1 earlierDate:date];
//获取其中更晚的日期
NSDate *lDate = [date1 laterDate:date];
//时间间隔
NSTimeInterval dateInterval = [date timeIntervalSinceDate:date1];
```

----

### 格式化`NSTimeInterval`

```objc
- (NSString *)stringFromTimeInterval:(NSTimeInterval)interval {
    NSInteger ti = (NSInteger)interval;
    NSInteger seconds = ti % 60;
    NSInteger minutes = (ti / 60) % 60;
    NSInteger hours = (ti / 3600);
    return [NSString stringWithFormat:@"%02ld:%02ld:%02ld", (long)hours, (long)minutes, (long)seconds];
}
```
