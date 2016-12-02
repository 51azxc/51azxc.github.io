title: "iOS UIImage使用"
date: 2015-04-10 14:24:48
tags: "image"
categories: ["iOS","Objective-C"]
---

### 获取图片

> [iphone学习笔记-UIImage读取图像资源](http://blog.csdn.net/gdmmhym/article/details/6616664)

```objc
UIImage *img = [[UIImage alloc]initWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"test" ofType:@"jpg"]];

//从网络获取图片
UIImage *img = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:@""]]];
```

### 缩放图片

> [The simplest way to resize an UIImage?](http://stackoverflow.com/questions/2658738/the-simplest-way-to-resize-an-uiimage)

```objc
-(UIImage *)imageWithImage:(UIImage *)image scaledToSize:(CGSize)size
{
    UIGraphicsBeginImageContextWithOptions(size, NO, 0.0);
    [image drawInRect:CGRectMake(0, 0, size.width, size.height)];
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}
```

### UIColor生成UIImage

> [UIColor生成UIImage](http://www.cocoachina.com/bbs/read.php?tid-91425.html)
> [Creating a UIImage from a UIColor to use as a background image for UIButton [duplicate]](http://stackoverflow.com/questions/6496441/creating-a-uiimage-from-a-uicolor-to-use-as-a-background-image-for-uibutton)

Objective-C:
```objc
- (UIImage *)createImageWithColor: (UIColor *)color andSize: (CGSize)size {
    CGRect rect = CGRectMake(0, 0, size.width, size.height);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}
```
Swift:
```swift
extension UIImage {
    static func create(color: UIColor, rect: CGRect) -> UIImage {
        UIGraphicsBeginImageContext(rect.size)
        let context = UIGraphicsGetCurrentContext()
        context!.setFillColor(color.cgColor)
        context!.fill(rect)
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return image!
    }
}

let imageRect = CGRect(x: 0, y: 0, width: 100, height: 100)
let image = UIImage.create(color: .black, rect: imageRect)
```