title: "iOS NSDictionary存整数"
date: 2015-04-10 17:42:32
tags: "collection"
categories: ["iOS","Objective-C"]
---

>来自[Passing NSInteger variable to NSMutableDictionary or NSMutableArray](http://stackoverflow.com/questions/1339394/passing-nsinteger-variable-to-nsmutabledictionary-or-nsmutablearray)

NSInteger不是一个对象，因此不能通过`[nsdictionary setObject: forKey:]`方法来存，这里需要把它转成`NSNumber`对象
```objc
NSString *temp = @"10";
NSNumber *tempNum = [NSNumber numberWithInteger: [temp integerValue]];
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
[dict setObject: tempNum forKey:@"temp"];
```