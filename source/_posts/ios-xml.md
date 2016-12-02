title: "IOS XML解析"
date: 2015-04-03 18:13:45
tags: "xml"
categories: ["iOS","Objective-C"]
---

转自[iOS开发小记：XML解析(GDataXML)][1]

----

IOS中自带了`NSXMLParser`可以解析XML，不过是基于SAX方式的，具体操作可以参考[NSXMLParser具体解析xml的应用详解][2]。而谷歌的`GDataXMLNode`则是基于DOM方式解析，支持XPath.不过`GDataXMLNode`已经很久没有更新了，如今需要使用的话，需要额外做一些工作
> 1. 在项目TARGETS General->Linked Frameworks and Libraries中，加入libxml2.dylib框架。
> 2. 在项目TARGETS Build Settings->Header Search Paths中，加入`${SDKROOT}/usr/include/libxml2`
> 3. 在项目TARGETS Build Phases->Compile Sources中选择`GDataXMLNode.m`后加入`-fno-objc-arc`，这样才能在ARC工程中使用非ARC框架。
> 4. 需要的类引入`GDataXMLNode.h`头文件

XML文件的结构为
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<note>
    <to>George</to>
    <from>John</from>
    <heading type="notification">eminder</heading>
    <body>Don't forget the meeting!</body>
</note>
```
解析代码为
```objc
//获取工程目录的xml文件
NSString *xmlPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"xml"];
NSData *xmlData = [[NSData alloc] initWithContentsOfFile:xmlPath];
    
GDataXMLDocument *doc = [[GDataXMLDocument alloc] initWithData:xmlData options:0 error:nil];
//获取根节点
GDataXMLElement *root = [doc rootElement];
//获得元素值
NSString *toStr = [[[root elementsForName:@"to"] firstObject] stringValue];
NSString *fromStr = [[[root elementsForName:@"from"] lastObject] stringValue];
    
GDataXMLElement *head = [[root elementsForName:@"heading"] objectAtIndex:0];
//获取元素节点属性值
NSString *typeStr = [[head attributeForName:@"type"] stringValue];
NSString *headStr = [head stringValue];
    
NSString *bodyStr = [[[root elementsForName:@"body"] lastObject] stringValue];
    
NSLog(@"to: %@, from: %@, head: %@, type: %@, body: %@",toStr,fromStr,headStr,typeStr,bodyStr);
```


----

[1]: http://www.isaced.com/post-199.html
[2]: http://blog.csdn.net/smking/article/details/8293566
['libxml/tree.h' file not found](http://blog.csdn.net/iitvip/article/details/9167649)























