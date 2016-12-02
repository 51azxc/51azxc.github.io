title: "iOS 照片视频"
date: 2015-12-28 10:53:45
tags: ["image", "photo"]
categories: ["iOS","Objective-C"]
---

### 照片
#### 裁剪照片

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

#### 从照片文件夹中获取图像资源写入到临时文件夹

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

#### 通过PHAsset获取资源文件大小

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

### 视频

> [Playback](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/02_Playback.html)

#### CMTimeMake

> [CMTimeMake和CMTimeMakeWithSeconds 详解](http://www.cnblogs.com/sell/archive/2013/01/29/2880832.html)

`CMTimeMake(a,b)` : a当前第几帧, b每秒钟多少帧.当前播放时间a/b
`CMTimeMakeWithSeconds(a,b)` : a当前时间,b每秒钟多少帧

#### 更新播放时间

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
