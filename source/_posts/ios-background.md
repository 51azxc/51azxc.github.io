title: "iOS零散知识收集"
date: 2015-04-03 16:43:35
tags: ["photo", "WebView"]
categories: ["iOS","Objective-C"]
---

收集一些平时在iOS开发中碰到的问题解决方法

<!-- more -->

### WKWebView中的Javascript交互

在**WKWebView**中，Javascript与OC的交互变的十分简单。只需要通过`WKScriptMessageHandler`代理中的`userContentController:didReceiveScriptMessage: `即可在网页中让Javascript发送消息给OC。

首先试着写一段js脚本
```javascript
var c=document.createElement('input');
c.type='checkbox'; 2.name='accept'; c2.id='accept';  c.checked=true;
var label = document.createElement('label');
label.htmlFor = 'accept';
label.appendChild(document.createTextNode('accept'));
document.getElementsByTagName('body')[0].appendChild(c);
document.getElementsByTagName('body')[0].appendChild(label);

var script = document.createElement('script');
script.innerHTML = \"
document.getElementById('accept').onclick = function toggle() {
    var obj = document.getElementById('accept');
    window.webkit.messageHandlers.accept.postMessage({'accept':obj.checked});
};
\";
document.getElementsByTagName('body')[0].appendChild(script);
```
这段js很简单，就是添加了一个`checkbox`元素，然后在点击事件中通过`postMessage`方法就可以在OC获取到js中传过来的对象。

接下来则是配置`WKUSErContentController`:
```objc
WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
configuration.userContentController = [WKUserContentController new];
//在文档末尾追加javascript
WKUserScript *script = [[WKUserScript alloc] initWithSource:scriptString injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];
[configuration.userContentController addUserScript:script];
//添加需要用作连接的对象
[configuration.userContentController addScriptMessageHandler:self name:@"accept"];
        
WKPreferences *preferences = [[WKPreferences alloc] init];
preferences.javaScriptEnabled = YES;
configuration.preferences = preferences;

self.webView =  [[WKWebView alloc] initWithFrame:self.view.frame configuration:configuration];
```
在这里通过`addScriptMEssageHandler`方法定义了一个js通知oc的对象accept，就是js中的`messageHandlers`后边声明的对象。

最后ViewController实现`WKScriptMessageHandler`代理，通过下列方法即可获得传输过来的数据：
```objc
# pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    NSLog(@"message name: %@, body: %@", message.name, message.body);
    //...
}
```
这里通过`message.name`可以获得传输对象的名称，就是上边定义的`accept`，而`message.body`则是获取到从js中传过来的数据

----

### iOS应用内切换语言

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

----

### 页面组件自适应导航栏

> [Status bar and navigation bar appear over my view's bounds in iOS 7](http://stackoverflow.com/questions/17074365/status-bar-and-navigation-bar-appear-over-my-views-bounds-in-ios-7)

当使用**Interface Builder**来构建页面的时候，如果是`UIScrollVIew`或者其子类`UITableView`的时候，当页面显示了`NavigationBar`的时候，`UIScrollVIew`部件会自动将坐标下移到`NavigationBar`下方，
这是因为`automaticallyAdjustsScrollViewInsets`属性为`YES`，如果不希望`scroll view`自动适应，将其设置为`NO`即可。

如果是普通的页面，一般都会直接拖到最顶层，如果这个时候页面显示了`NavigationBar`时，会将页面布局的一部分遮挡掉，以往的做法是将布局下移64，但是**iOS7**之后可以利用`edgesForExtendedLayout`来设置：
```objc
- (void)viewDidLoad {
    if ([self respondsToSelector:@selector(edgesForExtendedLayout)]) {
        self.edgesForExtendedLayout = UIRectEdgeLeft | UIRectEdgeRight | UIRectEdgeBottom;
    }
    ...
}
```

----

### 跳转页面之后背景半透明

> [用presentViewController一个背景颜色半透明的模态视图](http://www.cnblogs.com/oyhj/p/5120212.html)

当使用`presentViewController`方法弹出下一个`UIViewController`的时候，如果需要这个视图背景半透明，需要设置如下:
```
UIViewController *viewControllers = [UIViewController new];
self.definesPresentationContext = YES;
viewController.modalPresentationStyle = UIModalPresentationOverCurrentContext;
viewController.backgroudColor = [UIColor colorWithWhite: 0.1 alpha: 0.5];
//如果源视图不是NavigationController子视图，直接用self即可
[self.navigationController presentViewController:viewController animated:NO completion:nil];
```

----

### 隐藏状态栏

> [IOS7如何隐藏状态栏，貌似之前的没效果了](http://www.cocoachina.com/bbs/read.php?tid-153922.html)
> [How do I hide the status bar in a Swift iOS app?](http://stackoverflow.com/questions/24236912/how-do-i-hide-the-status-bar-in-a-swift-ios-app)

在需要隐藏状态栏的`ViewController`中重写`prefersStatusBarHidden`方法
```objc
- (BOOL)prefersStatusBarHidden {
    return YES;
}
```
然后在需要更改隐藏状态的地方调用`setNeedsStatusBarAppearanceUpdate`方法
```objc
[self setNeedsStatusBarAppearanceUpdate]
```

`swift`写法
```swift
override func prefersStatusBarHidden() -> Bool {
    return true
}
```

----

### 在应用中打开另一个应用

> [利用openURL，在IOS应用中打开另外一个应用](http://blog.sina.com.cn/s/blog_a170e5c80101gsdj.html)
> [canOpenUrl - This app is not allowed to query for scheme instragram iOS9](http://stackoverflow.com/questions/32870393/canopenurl-this-app-is-not-allowed-to-query-for-scheme-instragram-ios9)
> [iOS: Access app-info.plist variables in code](http://stackoverflow.com/questions/9530075/ios-access-app-info-plist-variables-in-code)

首先需要在`info.plist`注册自定义的`URL scheme`
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>test1</string>
        </array>
    </dict>
</array>
```
在`iOS9`以后，还需要添加以下属性:
```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>test2</string>
</array>
```
这里意为应用可以打开`scheme`为`test2`的应用。
然后在代码中可以使用`openUrl`函数打开应用：
```objc
//获取scheme
let prefix = (Bundle.main.object(forInfoDictionaryKey: "LSApplicationQueriesSchemes") as! NSArray)[0]
let url = URL(string: "\(prefix):client_id=test1&scoe=wopi&platform=iOS&app=test1&action=12345")
if UIApplication.shared.canOpenURL(url! as URL) {   //判断能否打开
    //打开应用
    UIApplication.shared.openURL(url! as URL)
} else {
    print("can not go to this app!")
}
```
接下来应用可以在`AppDelegate`类中的`handleOpenUrl`方法中获取到传过来的参数:
```objc
func application(_ application: UIApplication, handleOpen url: URL) -> Bool {
    print(url)
    
    let viewController = self.window?.rootViewController
    let message = url.absoluteString
    let alertController = UIAlertController.init(title: "url", message: message, preferredStyle: .alert)
    alertController.addAction(UIAlertAction.init(title: "OK", style: .default, handler: nil))
    viewController?.present(alertController, animated: true, completion: nil)
    
    return true;
}
```

----

### 在网页中打开应用

在网页`<script>`块中直接定义跳转的`scheme`为应用自定义的`scheme`即可

```js
//如果无法打开应用则跳转到AppStore中
setTimeout(function () {
    window.location = "https://itunes.apple.com/cn/app/microsoft-word/id586447913"
}, 25);
window.location = "test1://";
```

----

### UIButton 内容左对齐

> [How to set the title of UIButton as left alignment?](http://stackoverflow.com/questions/2765024/how-to-set-the-title-of-uibutton-as-left-alignment)

设置`contentHorizontalAlignment`属性
```objc
[button setContentHorizontalAlignment: UIControlContentHorizontalAlignmentLeft];
```
或者使用`UIEdgeInsetsMake(top, left, bottom, right)`方法来设置缩进，正数为缩进，负数为突出
```objc
[button setContentEdgeInsets: UIEdgeInsetsMake(0, -20, 0, 0)];
```

----

### 裁剪照片

> [How to crop an image from AVCapture to a rect seen on the display](http://stackoverflow.com/questions/15951746/how-to-crop-an-image-from-avcapture-to-a-rect-seen-on-the-display)

通过`AVCaptureSession`以及`AVCaptureStillImageOuput`获取的照片默认填充整个屏幕，与自定义的显示屏幕并不相同，因此需要裁剪照片至所见区域

```objc
- (UIImage *)cropImage: (UIImage *)image {
    //获取需要裁剪的矩形
    CGRect outputRect = [self.captureVideoPreviewLayer metadataOutputRectOfInterestForRect:self.captureVideoPreviewLayer.bounds];
    CGImageRef takenCGImage = image.CGImage;
    size_t width = CGImageGetWidth(takenCGImage);
    size_t height = CGImageGetHeight(takenCGImage);
    CGRect cropRect = CGRectMake(outputRect.origin.x * width, outputRect.origin.y * height, outputRect.size.width * width, outputRect.size.height * height);
    
    CGImageRef cropCGImage = CGImageCreateWithImageInRect(takenCGImage, cropRect);
    //需要使用原图的朝向
    UIImage *croppedImage = [UIImage imageWithCGImage:cropCGImage scale:image.scale orientation:image.imageOrientation];
    CGImageRelease(cropCGImage);
    return croppedImage;
}
```

----

### 从照片文件夹中获取图像资源写入到临时文件夹

> [ALAsset , send a photo to a web service including its exif data](http://stackoverflow.com/questions/6881923/alasset-send-a-photo-to-a-web-service-including-its-exif-data)

```objc
ALAsset *selectedAsset = [self.selectAssets objectForKey:key];
if (selectedAsset) {
    long long byteSize = selectedAsset.defaultRepresentation.size;
    NSMutableData *rawData = [[NSMutableData alloc] initWithCapacity:byteSize];
    void *bufferPointer = [rawData mutableBytes];
    NSError *error = nil;
    [selectedAsset.defaultRepresentation getBytes:bufferPointer fromOffset:0 length:byteSize error:&error];
    if (error) {
        NSLog(@"get asset data error: %@",error);
    }
    rawData = [NSMutableData dataWithBytes:bufferPointer length:byteSize];
    [cloudObject setFilesize:[NSString stringWithFormat:@"%lld", byteSize]];
    
    NSString *filePath = [NSTemporaryDirectory() stringByAppendingPathComponent:fileName];
    if ([rawData writeToFile:filePath atomically:YES]) {
        [cloudObject setTempFilePath:filePath];
        NSLog(@"current file path: %@", filePath);
    }
}
```

----

### 通过PHAsset获取资源文件大小

> [How can I determine file size on disk of a video PHAsset in iOS8](http://stackoverflow.com/questions/26549938/how-can-i-determine-file-size-on-disk-of-a-video-phasset-in-ios8)

**iOS8**之后，`ALAsset`被标记为不推荐，取而代之的是`PHAsset`。如果要获取文件大小，不能使用`asset.defaultRepresentation.size`了，需要用到以下方法:
```objc
PHAsset *asset = [[PHAsset fetchAssetsWithLocalIdentifiers:@[cloudObject.tempFilePath] options:nil] firstObject];
if (asset.mediaType == PHAssetMediaTypeImage) {
    PHImageRequestOptions *imageOptions = [[PHImageRequestOptions alloc] init];
    imageOptions.deliveryMode = PHImageRequestOptionsDeliveryModeHighQualityFormat;
    imageOptions.resizeMode = PHImageRequestOptionsResizeModeExact;
    imageOptions.synchronous = YES;
    imageOptions.networkAccessAllowed = NO;
    [[PHImageManager defaultManager] requestImageDataForAsset:asset options:imageOptions resultHandler:^(NSData * _Nullable imageData, NSString * _Nullable dataUTI, UIImageOrientation orientation, NSDictionary * _Nullable info) {
       NSLog(@"length %f",imageData.length/(1024.0*1024.0));
    }];
} else if (asset.mediaType == PHAssetMediaTypeVideo) {
    PHVideoRequestOptions *videoOptions = [[PHVideoRequestOptions alloc] init];
    videoOptions.version = PHVideoRequestOptionsVersionOriginal;
    videoOptions.networkAccessAllowed = NO;
    [[PHImageManager defaultManager] requestAVAssetForVideo:asset options:videoOptions resultHandler:^(AVAsset * _Nullable asset, AVAudioMix * _Nullable audioMix, NSDictionary * _Nullable info) {
        
        if ([asset isKindOfClass:[AVURLAsset class]]) {
            AVURLAsset *urlAsset = (AVURLAsset *)asset;
            NSNumber *size;
            [urlAsset.URL getResourceValue:&size forKey:NSURLFileSizeKey error:nil];
            NSLog(@"size is %f",[size floatValue]/(1024.0*1024.0));
            NSData *data = [NSData dataWithContentsOfURL:urlAsset.URL];
            NSLog(@"length %f",[data length]/(1024.0*1024.0));
        }
    }];
}
```
不要忘了引入框架`@import Photos`

----

### CMTimeMake

> [CMTimeMake和CMTimeMakeWithSeconds 详解](http://www.cnblogs.com/sell/archive/2013/01/29/2880832.html)

`CMTimeMake(a,b)` : a当前第几帧, b每秒钟多少帧.当前播放时间a/b
`CMTimeMakeWithSeconds(a,b)` : a当前时间,b每秒钟多少帧

### 更新播放时间

使用`addPeriodicTimeObserverForInterval:queue:usingBlock`方法可以监听到播放时间的变化

```objc
__weak typeof(self) weakSelf = self;
 id playerObserver = [self.player addPeriodicTimeObserverForInterval:CMTimeMakeWithSeconds(1.0, NSEC_PER_SEC) queue:NULL usingBlock:^(CMTime time) {
    Float64 interval = CMTimeGetSeconds(time);
    NSInteger seconds = (NSInteger)interval % 60;
    NSInteger minutes = ((NSInteger)interval / 60) % 60;
    NSInteger hours = (NSInteger)interval / 3600;
    NSString *currentTime = [NSString stringWithFormat:@"%02ld:%02ld:%02ld", (long)hours, (long)minutes, (long)seconds];
    weakSelf.timeView.text = currentTime;
}];
```
使用完需要移除观察者`[self.player removeTimeObserver: playerObserver]`
