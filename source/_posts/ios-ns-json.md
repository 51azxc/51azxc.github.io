title: "利用NSJSONSerialization操作JSON对象"
date: 2015-04-03 18:27:31
tags: "json"
categories: ["iOS","Objective-C"]
---

> [NSJSONSerialization介绍](http://blog.csdn.net/uxyheaven/article/details/7888559)
> [iOS NSDictionary、NSData、JSON数据类型相互转换](http://blog.csdn.net/x1135768777/article/details/8529297)
> [IOS5 JSON](http://www.cnblogs.com/cokecoffe/archive/2012/06/02/2537104.html)

NSArray,NSDictionary转成JSON
```objc
//判断是否能转成json对象
if ([NSJSONSerialization isValidJSONObject:array]) {
  NSError *error;
  NSData *jsonData = [NSJSONSerialization dataWithJSONObject:array options:NSJSONWritingPrettyPrinted error:&error];
}
```

JSON对象转NSArray，NSDictionary
```objc
NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:&error];
```

----

### IOS JSON格式转model

> [JSON与MODEL互转](http://blog.csdn.net/woaifen3344/article/details/39301203)
> [Loop through all object properties at runtime](http://stackoverflow.com/questions/9269372/loop-through-all-object-properties-at-runtime)


```objc
+(id)dictToModel: (NSDictionary *)dict WithClassName: (NSString *)className
{
    if(dict == nil || className == nil || className.length == 0){
        return nil;
    }
    //根据类名获取相关类引用
    id model = [[NSClassFromString(className) alloc] init];
    id classObject = objc_getClass([className UTF8String]);
    
    unsigned int count = 0;
    //获取类中属性
    objc_property_t *pros = class_copyPropertyList(classObject, &count);
    Ivar *ivars = class_copyIvarList(classObject, nil);
    
    for (int i=0; i<count; i++) {
        NSString *memberName = [NSString stringWithUTF8String:ivar_getName(ivars[i])];
        const char *type = ivar_getTypeEncoding(ivars[i]);
        NSString *dataType = [NSString stringWithCString:type encoding:NSUTF8StringEncoding];
        
        for (int j=0; j<count; j++) {
            objc_property_t property = pros[j];
            NSString *propertyName = [[NSString alloc] initWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            
            if ([memberName isEqualToString:propertyName]) {
                id propertyValue = [dict objectForKey:propertyName];
                //int类型
                if ([dataType hasPrefix:@"i"]){
                    int tmp = [[NSString stringWithFormat:@"%@", propertyValue] intValue];
                    propertyValue = [NSNumber numberWithInt:tmp];
                }else{
                    //NSString类型
                    if ([dataType rangeOfString:@"NSString"].location!=NSNotFound) {
                        if([propertyValue rangeOfString:@"&#39;"].location!=NSNotFound){
                            propertyValue = [propertyValue stringByReplacingOccurrencesOfString:@"&#39;" withString:@"'"];
                        }
                        if([propertyValue rangeOfString:@"&amp;"].location!=NSNotFound){
                            propertyValue = [propertyValue stringByReplacingOccurrencesOfString:@"&amp;" withString:@"&"];
                        }
                        if([propertyValue rangeOfString:@"&quot;"].location!=NSNotFound){
                            propertyValue = [propertyValue stringByReplacingOccurrencesOfString:@"&quot;" withString:@""];
                        }
                    }
                    //NSArray类型
                    NSRange subRange = [dataType rangeOfString:@"NSArray"];
                    if (subRange.location != NSNotFound) {
                        NSArray *array = [NSArray arrayWithArray:propertyValue];
                        NSMutableArray *mutableArray = [[NSMutableArray alloc] init];
                        for (NSDictionary *subDict in array) {
                            NSString *subClassName = [@"Eye" stringByAppendingString:[memberName capitalizedString]];
                            [mutableArray addObject: [self dictToModel:subDict WithClassName:subClassName]];
                        }
                        propertyValue = mutableArray;
                    }
                }
                //将相关属性值放入到model中
                [model setValue:propertyValue forKey:memberName];
                break;
            }else{
                continue;
            }
            
        }
    }
    //释放内存    
    free(pros);
    free(ivars);

    return model;

}
```
