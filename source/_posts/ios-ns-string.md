title: "NSString 相关练习"
date: 2015-04-02 15:04:12
tags: ["string", "collection"]
categories: ["iOS","Objective-C"]
---

> [NSString拼接字符串](http://blog.csdn.net/snn1410/article/details/7527078)
> [IOS NSString 截取，objectAtIndex，rangeOfString，stringWithContentsOfFile，NSEnumerator](http://www.cnblogs.com/csj007523/archive/2012/07/15/2592302.html)
> [OBC: NSString 与 NSArray 互转](http://laiguowei2004.blog.163.com/blog/static/3682900020139297315510/)
> [iOS NSString 和NSData 转换](http://fei263.blog.163.com/blog/static/9279372420115125731356/)
> [NSString  变大写，小写，每个字母开头大写](http://blog.sina.com.cn/s/blog_8345c9c90100vj6w.html)
> [NSString的几种常用方法](http://www.cnblogs.com/superhappy/archive/2012/11/19/2778084.html)
> [NSString to CFStringRef and CFStringRef to NSString in ARC?](http://stackoverflow.com/questions/17227348/nsstring-to-cfstringref-and-cfstringref-to-nsstring-in-arc)
> [Objective-C: Extract filename from path string](http://stackoverflow.com/questions/1098957/objective-c-extract-filename-from-path-string)
> [Objective-C中trim的实现](http://blog.csdn.net/yhawaii/article/details/7871784)

#### 截取制定位置之后的字符串

```objc
NSString *str = @"file:///Users/admin/Documents/test.html";
NSString *s1 = [str substringFromIndex:5];
NSLog(@"s1: %@",s1);
```

#### 截取指定下标之前的字符串

```objc
NSString *s2 = [str substringToIndex:5];
NSLog(@"s2: %@",s2);
```

#### 截取指定范围的字符串

```objc
NSRange range = [str rangeOfString:@"admin"];
NSString *s3 = [str substringWithRange:range];
NSLog(@"s3: %@",s3);
//从反方向截取字符串
NSRange range1 = [str rangeOfString:@"/" options:NSBackwardsSearch];
if (range.location!=NSNotFound) {
  NSString *s3_1 = [str substringFromIndex:range1.location];
  NSLog(@"s3.1: %@",s3_1);
}
```

#### 字符串数组互转

* 字符串转数组

```objc
NSArray *array1 = [str componentsSeparatedByString:@"/"];
for(NSString *array_str in array1){
    NSLog(@"array_str: %@",array_str);
}
```

* 数组转字符串

```objc
NSString *str_array = [array1 componentsJoinedByString:@"/"];
NSLog(@"str_array: %@",str_array);
```

#### 拼接字符串

```objc
NSString *s4_1 = [[NSString alloc] initWithFormat:@"%@%@",str,@"?a=1"];
NSString *s4_2 = [str stringByAppendingString:@"?a=1"];
NSString *s4_3 = [str stringByAppendingFormat:@"%@%@",@"?",@"a=1"];
NSLog(@"s4.1: %@\ns4.2: %@\ns4.3: %@",s4_1, s4_2, s4_3);
```

#### CGFloat转NSString

```objc
NSString *temp = [NSString stringWithFormat:@"x:%f,y:%g",1.1,2.2];
NSLog(@"temp: %@",temp);
```

#### 大小写及开头字母大写

```objc
NSLog(@"upper: %@\nlower: %@\ncap: %@",[str uppercaseString],[str lowercaseString],[str capitalizedString]);
```

#### NSString NSData互转

```objc
//NSString 转 NSData
NSData *data = [str dataUsingEncoding:NSUTF8StringEncoding];
//NSData 转 NSString
NSString *s5 = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
NSLog(@"s5: %@",s5);
```

#### 替换

```objc
//全部替换
NSString *s6_1 = [str stringByReplacingOccurrencesOfString:@"/" withString:@"-"];
NSLog(@"s6: %@",s6_1);
```

#### 其他

```objc
NSString *cs1 = @"This is a String1";
NSString *cs2 = @"This is a String2";
//判断两者是否相同
BOOL r1 = [cs1 compare:cs2] == NSOrderedSame;
//判断对象值的大小
BOOL r2 = [cs1 compare:cs2] == NSOrderedAscending;
//不考虑大小写
BOOL r3 = [cs1 caseInsensitiveCompare:cs2] == NSOrderedSame;
NSLog(@"same: %i, asc: %i, same: %@",r1,r2,r3?@"YES":@"NO");
        
//删除部分字符串元素
NSMutableString *nstr = [NSMutableString stringWithString:str];
[nstr deleteCharactersInRange:[str rangeOfString:@"admin"]];
NSLog(@"delete: %@",nstr);
//插入字符串
[nstr insertString:@"admin" atIndex:10];
NSLog(@"insert: %@",nstr);
```

#### NSString CFStringRef互转

```objc
NSString *yourFriendlyNSString = (__bridge NSString *)yourFriendlyCFString;
CFStringRef yourFriendlyCFString = (__bridge CFStringRef)yourFriendlyNSString;
```

#### 文件名相关操作

```objc
//添加路径
NSString *filePath = [NSTemporaryDirectory() stringByAppendingPathComponent: @"temp.txt"];
//增加相关后缀名
NSString *fileName = [@"temp" stringByAppendingPathExtension:@"txt"];
//通过路径获取文件名
//lastPathComponent获取到temp.txt,
//stringByDeletingPathExtension获取到temp
NString *fileName1 = [[string lastPathComponent] stringByDeletingPathExtension];
//获取后缀
NSString extension = [[string lastPathComponent] pathExtension];
```

----

#### 去除首尾空格

```objc
[@" abc " stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
```
