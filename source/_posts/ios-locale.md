title: "iOS应用内切换语言"
date: 2016-01-06 12:24:04
tags:
categories: ["iOS","Objective-C"]
---

> [How to force NSLocalizedString to use a specific language](http://stackoverflow.com/questions/1669645/how-to-force-nslocalizedstring-to-use-a-specific-language)
> [ios开发应用内实现多语言自由切换](http://www.cocoachina.com/bbs/read.php?tid=149950)

使用系统内置的`NSLocalizedString(key,comment)`函数即可获得对应的本地化语句。如果需要在应用内切换其他语言，可以自行添加一个定义
```objc
#define CustomLocalizedString(key, comment) \
[[NSBundle bundleWithPath: [[NSBundle mainBundle] pathForResource: [NSString stringWithFormat:@"%@",[[[NSUserDefaults standardUserDefaults] objectForKey:@"AppleLanguages"] firstObject]] ofType:@"lproj"]] localizedStringForKey:(key) value:@"" table:nil]
```
然后切换语言需要修改`AppleLanguages`中的数组数据位置
```objc
[[NSUserDefaults standardUserDefaults] setObject:[NSArray arrayWithObjects:@"en-US", @"zh-Hant", @"zh-Hans", nil] forKey:@"AppleLanguages"];
[[NSUserDefaults standardUserDefaults] synchronize];
```
将需要切换的语言放置到第一个元素中即可