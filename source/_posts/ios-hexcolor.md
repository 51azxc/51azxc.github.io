title: "iOS Hex Color转UIColor"
date: 2015-12-24 11:19:23
tags: ["color"]
categories: ["iOS","Objective-C"]
---

> [How can I create a UIColor from a hex string?](http://stackoverflow.com/questions/1560081/how-can-i-create-a-uicolor-from-a-hex-string?page=1&tab=votes#tab-top)
> [Objective C parse hex string to integer](http://stackoverflow.com/questions/3648411/objective-c-parse-hex-string-to-integer)

首先利用`NSScanner`类将传入的`NSString`类型的参数转成16进制的整数，然后再取得对应的RGB数值

```objc
- (UIColor *)getUIColorWithHexColor: (NSString *)hexColor andAlpha: (CGFloat)alpha {
    unsigned hex = 0;
    NSScanner *hexScanner = [NSScanner scannerWithString:hexColor];
    if ([hexColor hasPrefix:@"#"]) {
        // pass '#' character
        [hexScanner setScanLocation:1];
    }
    [hexScanner scanHexInt:&hex];
    return [UIColor colorWithRed: ((float)((hex & 0xFF0000) >> 16))/255.0
                           green: ((float)((hex & 0x00FF00) >>  8))/255.0
                            blue: ((float)((hex & 0x0000FF) >>  0))/255.0
                           alpha: alpha];
}
```
当然还可以通过**[这个网站](http://uicolor.xyz/#/hex-to-ui)**来直接取得数值
