title: "NSError相关"
date: 2015-04-10 18:06:56
tags:
categories: ["iOS","Objective-C"]
---

### 自定义NSError

> [iphone跬步之－－错误信息 NSError](http://www.cnblogs.com/xiaodao/archive/2012/07/04/2576292.html)


```objc
NSString *errorMsg = [NSString stringWithFormat:@"%d",statusCode];
        NSDictionary *userInfo = [NSDictionary dictionaryWithObject:errorMsg forKey:NSLocalizedDescriptionKey];
        NSError *httpError = [NSError errorWithDomain:@"eyePadMini.Model.JsonToModel" code:statusCode userInfo:userInfo];
```

`domain`可以随意指定，`code`为错误标识，系统的code一般都大于零，自定code可以用枚举（最好用负数），userInfo自定义错误信息，NSLocalizedDescriptionKey是NSError头文件中预定义的键，标识错误的本地化描述。可以通过NSError的localizedDescription方法获得对应的值信息
```objc
NSLog(@"%@",[httpError localizedDescription]);
```

###### 调用函数获取NSError
如果调用了一个函数其中会产生`NSError`对象，可以在方法中添加`NSError`的指针的指针
```objc
+(NSData *)getDataFromURL: (NSString *)url withError: (NSError **)e
{
    NSError *error = nil;
    *e = error;
}
```
然后在调用方法的地方就可以得到返回的错误信息

----

> [How can I use NSError in my iPhone App?](http://stackoverflow.com/questions/4654653/how-can-i-use-nserror-in-my-iphone-app)

```objc
+(NSData *)getDataFromURL: (NSString *)url withError: (NSError **)e
{
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    __block NSMutableData *result = [[NSMutableData alloc] init];
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDataTask *dataTask = [session dataTaskWithURL:[NSURL URLWithString:url] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSHTTPURLResponse *resp = (NSHTTPURLResponse *)response;
        NSInteger statusCode = [resp statusCode];
        if (error == nil && statusCode == 200) {
            result = [NSMutableData dataWithData:data];
        }else if(error != nil){
            *e = error;
            NSLog(@"NSURLSessionDataTask Error: %@", [error localizedDescription]);
        }else if(statusCode != 200){
            NSString *errorMsg = [NSString stringWithFormat:@"%d",statusCode];
            NSDictionary *userInfo = [NSDictionary dictionaryWithObject:errorMsg forKey:NSLocalizedDescriptionKey];
            NSError *httpError = [NSError errorWithDomain:@"eyePadMini.Model.JsonToModel" code:statusCode userInfo:userInfo];
            *e = httpError;
        }
        dispatch_semaphore_signal(semaphore);
    }];
    [dataTask resume];
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    return result;

}

```
