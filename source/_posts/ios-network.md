title: "iOS 网络框架"
date: 2015-04-13 18:15:52
tags: "NSURLSession"
categories: ["iOS","Objective-C"]
---

### NSURLSession
#### 基本使用

> [NSURLSession——网络框架新生代](http://zhaoxuefeng.gitcafe.io/2014/08/03/nsurlsession3/)
> [iOS 7 SDK：后台传输服务](http://www.cocoachina.com/industry/20131120/7380.html)
> [Networking with NSURLSession: Part 1](http://code.tutsplus.com/tutorials/networking-with-nsurlsession-part-1--mobile-21394)
> [NSURLSession with NSBlockOperation and queues](http://stackoverflow.com/questions/21198404/nsurlsession-with-nsblockoperation-and-queues)
> [How to get data to return from NSURLSessionDataTask](http://stackoverflow.com/questions/20871506/how-to-get-data-to-return-from-nsurlsessiondatatask)
> [If url exists Objective-c](http://stackoverflow.com/questions/2985229/if-url-exists-objective-c)

使用代理模式
```objc
@interface Download : NSObject<NSURLSessionDownloadDelegate, NSURLSessionTaskDelegate, NSURLSessionDelegate>

@property(nonatomic) NSURLSession *session;
@property(nonatomic) NSURLSessionDownloadTask *sessionTask;

-(void)startDownload;

@end

-(void)startDownload
{
    //默认的配置
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    self.session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];
    NSURL *downloadURL = [NSURL URLWithString:downloadURLString];
    NSURLRequest *requset = [NSURLRequest requestWithURL:downloadURL];
    self.sessionTask = [self.session downloadTaskWithRequest:requset];
    //调用resume方法才会开始启动网络连接
    [self.sessionTask resume];
}
//下载成功后触发的方法
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
    
    NSFileManager *fileMgr = [NSFileManager defaultManager];
    NSArray *urls = [fileMgr URLsForDirectory:NSCachesDirectory inDomains:NSUserDomainMask];
    NSURL *url = [urls objectAtIndex:0];
    NSURL *origionURL = [[downloadTask originalRequest] URL];
    NSURL *destinationPath = [url URLByAppendingPathComponent:[origionURL lastPathComponent]];
    NSError *error = nil;
    [fileMgr removeItemAtURL:destinationPath error:&error];
    BOOL success = [fileMgr copyItemAtURL:location toURL:destinationPath error:&error];
    if (success) {
        NSLog(@"location: %@",location);
        NSLog(@"destinationPath: %@",destinationPath);
    }else{
        NSLog(@"Error during the copy: %@", [error localizedDescription]);
    }
}
//如果出现错误时调用的方法
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error
{
    if (error == nil) {
        NSLog(@"Task: %@ completed successfully", task);
    }else{
        NSLog(@"Task: %@ completed with error: %@", task, [error localizedDescription]);
    }
    double progress = (double) task.countOfBytesReceived / (double)task.countOfBytesExpectedToReceive;
    NSLog(@"progress: %f",progress);
    self.sessionTask = nil;
}
//下载过程中得调用方法
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    if (downloadTask == self.sessionTask) {
        double progress = (double) totalBytesWritten / (double) totalBytesExpectedToWrite;
        NSLog(@"DownloadTask: %@ progress: %1f", downloadTask, progress);
        
    }
}

```

或者使用`block`模式

```objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
__block NSMutableData *result = [[NSMutableData alloc] init];
NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *dataTask = [session dataTaskWithURL:[NSURL URLWithString:url] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    NSString *result = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"result: %@",result);
    dispatch_semaphore_signal(semaphore);
}];
[dataTask resume];
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
//防止内存泄漏
[session finishTasksAndInvalidate];
```

----

##### 缓存策略

> [iOS开发之缓存（一）：内存缓存](http://www.cnblogs.com/zhuqil/archive/2011/07/30/2122127.html)
> [ios url缓存策略——NSURLCache、 NSURLRequest、Http规则](http://blog.sina.com.cn/s/blog_6f9a971801018abb.html)
> [Why is NSURLSession don't use my configured NSURLCache?](http://stackoverflow.com/questions/20768968/why-is-nsurlsession-dont-use-my-configured-nsurlcache)

URL缓存策略
```objc
requestWithURL:cachePolicy:timeoutInterval:
```
1. NSURLRequestUseProtocolCachePolicy NSURLRequest默认的cache policy，使用Protocol协议定义。
2. NSURLRequestReloadIgnoringCacheData 忽略缓存直接从原始地址下载。
3. NSURLRequestReturnCacheDataElseLoad 只有在cache中不存在data时才从原始地址下载。
4. NSURLRequestReturnCacheDataDontLoad 只使用cache数据，如果不存在cache，请求失败；用于没有建立网络连接离线模式；
5. NSURLRequestReloadIgnoringLocalAndRemoteCacheData：忽略本地和远程的缓存数据，直接从原始地址下载，与NSURLRequestReloadIgnoringCacheData类似。
6. NSURLRequestReloadRevalidatingCacheData:验证本地数据与远程数据是否相同，如果不同则下载远程数据，否则使用本地数据。

----

#### 出现*kCFStreamErrorDomainSSL, -9802 when connecting to a server by IP address*错误的解决方案

> [kCFStreamErrorDomainSSL, -9802 when connecting to a server by IP address through HTTPS in iOS 9](http://stackoverflow.com/questions/30778579/kcfstreamerrordomainssl-9802-when-connecting-to-a-server-by-ip-address-through)

*iOS9*开始需要网络服务器采取更高端智能安全的网络连接协议，因此需要将HTTP协议换成HTTPS协议，不然会出错，如果不愿意换，可以在项目info.list文件添加如下代码:
```xml
<key>NSAppTransportSecurity</key>
<dict>
  <!--Include to allow all connections (DANGER)-->
  <key>NSAllowsArbitraryLoads</key>
      <true/>
</dict>
```

----

#### 判断网络连接

> [IOS检测网络连接状态(转)](http://www.cnblogs.com/mrhgw/archive/2012/08/01/2617760.html)

利用在`Reachability`的相关方法在`AppDelegate`中的`applicationDidFinishLaunching`方法中判别网络情况。
```objc
- (void) applicationDidFinishLaunching:(UIApplication *)application
{
    // 监测网络情况
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(reachabilityChanged:)
                                                 name: kReachabilityChangedNotification
                                               object: nil];
    Reachability *hostReach = [Reachability reachabilityWithHostName: @"www.baidu.com"];
    [hostReach startNotifier];
}

- (void)reachabilityChanged:(NSNotification *)note {
    Reachability* curReach = [note object];
    NSParameterAssert([curReach isKindOfClass: [Reachability class]]);
    NetworkStatus status = [curReach currentReachabilityStatus];
    //NotReachable-无网络，ReachableViaWWAN-3G网络，ReachableViaWiFi-WIFI网络
    if (status == NotReachable) {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"AppName"
                                                        message:@"NotReachable"
                                                       delegate:nil
                                              cancelButtonTitle:@"YES" otherButtonTitles:nil];
        [alert show];
    }
}
```

----


#### URLWithString方法返回nil

> [URLWithString: returns nil](http://stackoverflow.com/questions/1981390/urlwithstring-returns-nil)

当`String`中含有中文的时候，使用[NSURL URLWithString: str]方法得到的`NSURL`对象为`nil`。为了防止这种情况出现，需要先将`String`对象进行转码:
```objc
NSURL *url = [NSURL URLWithString: [str stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding]];
```
这样能够预防`NSURL`对象返回为`nil`的问题。