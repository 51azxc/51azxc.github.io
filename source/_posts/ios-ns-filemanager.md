title: "NSFileManager文件操作"
date: 2015-04-03 14:11:18
tags: "io"
categories: ["iOS","Objective-C"]
---

> [iOS学习笔记（十七）——文件操作（NSFileManager）](http://blog.csdn.net/xyz_lmn/article/details/8968213)
> [iOS沙盒目录结构解析](http://blog.csdn.net/wzzvictory/article/details/18269713)
> [iOS中对文件的操作 (NSSearchPathForDirectoriesInDomains)](http://blog.csdn.net/happytengfei/article/details/8011886)


###### 文件目录
- **Documents**: 可以将程序产生的数据文件保存在此目录中。iTunes同步。 
- **Library**: 存放默认设置或其它状态信息。iTunes同步(除了子目录Caches)。
- **Library/Caches**: 存放程序产生的缓存文件，例如网络缓存之类。iTunes不同步。
- **Library/Preferences**：应用程序的偏好设置文件。我们使用NSUserDefaults写的设置数据都会保存到该目录下的一个plist文件中，即.plist文件。iTunes同步。
- **tmp**: 存放临时文件。系统会自动清除目录下的文件。iTunes不同步。
###### 获取主要目录方法
```objc
//获取根目录
NSLog(@"home: %@",NSHomeDirectory());
//获取tmp目录
NSLog(@"tmp: %@",NSTemporaryDirectory());
//获取Documents目录
NSLog(@"doc: %@",[NSSearchPathForDirectoriesInDomains(NSDocumentationDirectory, NSUserDomainMask, YES) objectAtIndex:0]);
//获取Library目录
NSLog(@"lib: %@",[NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES) objectAtIndex:0]);
//获取Cache目录
NSLog(@"cache: %@",[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) objectAtIndex:0]);
```
主要使用方法
```objc
[NSSearchPathForDirectoriesInDomains(<#NSSearchPathDirectory directory#>, <#NSSearchPathDomainMask domainMask#>, <#BOOL expandTilde#>)]
```
此方法有三个参数：
- **directory**：NSSearchPathDirectory类型的enum值，表明我们要搜索的目录名称，比如这里用NSDocumentDirectory表明我们要搜索的是Documents目录。如果我们将其换成NSCachesDirectory就表示我们搜索的是Library/Caches目录。
- **domainMask**：NSSearchPathDomainMask类型的enum值，指定搜索范围，这里的NSUserDomainMask表示搜索的范围限制于当前应用的沙盒目录。还可以写成NSLocalDomainMask（表示/Library）、NSNetworkDomainMask（表示/Network）等。
- **expandTilde**：BOOL值，表示是否展开波浪线~。在iOS中~的全写形式是/User/userName，该值为YES即表示写成全写形式，为NO就表示直接写成“~”。

###### 文件操作
```objc
NSString *cacheDir = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) objectAtIndex:0];
NSFileManager *mgr = [NSFileManager defaultManager];
//创建目录
NSString *createDir = [cacheDir stringByAppendingPathComponent:@"test"];
if ([mgr createDirectoryAtPath:createDir withIntermediateDirectories:YES attributes:nil error:nil]) {
    NSLog(@"成功创建文件夹");
}else{
    NSLog(@"创建文件夹失败");
}
//创建文件
NSString *createFile = [cacheDir stringByAppendingPathComponent:@"json.txt"];
if ([mgr createFileAtPath:createFile contents:nil attributes:nil]) {
    NSLog(@"成功创建文件");
}else{
    NSLog(@"创建文件失败");
}
//删除文件
[mgr removeItemAtPath:createFile error:nil];
//获取文件属性
NSDictionary *attrDict = [mgr attributesOfItemAtPath:createFile error:nil];
//获取文件大小
NSLog(@"file size: %@", [attrDict objectForKey: NSFileSize]);
```