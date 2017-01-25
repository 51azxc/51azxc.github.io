title: "iOS UIView相关知识"
date: 2015-04-10 10:26:31
tags:
categories: ["iOS","Objective-C"]
---

### UIView 设置部分圆角

> [iOS开发之指定UIView的某几个角为圆角](http://sjh787291806.blog.163.com/blog/static/21396319620131015105856616/)

----

```objc
//设置左上/右上角为圆角
UIBezierPath *usrPath = [UIBezierPath bezierPathWithRoundedRect:usrView.bounds byRoundingCorners:UIRectCornerTopLeft|UIRectCornerTopRight cornerRadii:CGSizeMake(5.0f, 5.0f)];
CAShapeLayer *usrLayer = [[CAShapeLayer alloc] init];
usrLayer.frame = usrView.bounds;
usrLayer.path = usrPath.CGPath;
usrView.layer.mask = usrLayer;
```

`UIRectCorner`类型主要有以下参数

* `UIRectCornerTopLeft`
* `UIRectCornerTopRight`
* `UIRectCornerBottomLeft`
* `UIRectCornerBottomRight`
* `UIRectCornerAllCorners`

可以指定各个角的圆角属性

----

### 使subview居中

> [How to center a subview of UIView](http://stackoverflow.com/questions/11251988/how-to-center-a-subview-of-uiview)

```objc
subView.center = CGPointMake(self.view.frame.size.width  / 2, self.view.frame.size.height / 2);
// or
child.center = [parent convertPoint:parent.center fromView:parent.superview];
```

----

### 删除所有的子视图

> [Remove all subviews](http://stackoverflow.com/questions/2156015/remove-all-subviews)

```objc
[[someUIView subviews]
 makeObjectsPerformSelector:@selector(removeFromSuperview)];
```

不管用时只能使用遍历方法逐个删除
```objc
NSArray *viewsToRemove = [self.view subviews];
for (UIView *v in viewsToRemove) {
    [v removeFromSuperview];
}
```

如果需要删除还存在于视图中的子视图需要判断一下:
```objc
if ([view isDescendantOfView:superview]) {
    [view removeFromSuperview];
}
````

----

### 更改视图大小

> [CGRectInset CGRectoffset UIEdgeInsetsInsetRect 这三个函数的使用情况](http://blog.csdn.net/ys410900345/article/details/42924827)

```objc
CGRect rect = CGRectMake(100, 100, 100, 100);
//将原来的矩形放大或者缩小，正数表示放大，负数表示缩小
CGRect insetRect = CGRectInset(rect, 10, -10);
NSLog(@"Inset rect x: %.0f, y: %.0f, w: %.0f, h: %.0f", insetRect.origin.x, insetRect.origin.y, insetRect.size.width, insetRect.size.height);
//将原来的矩形变换位置
CGRect offsetRect = CGRectOffset(rect, 20, -20);
NSLog(@"Offset rect: %@", [NSValue valueWithCGRect:offsetRect]);
//在原来的矩形基础上内切一个矩形
CGRect edgeInsetRect = UIEdgeInsetsInsetRect(rect, UIEdgeInsetsMake(-10, -10, -30, -40));
NSLog(@"Edge inset rect: %@", [NSValue valueWithCGRect:edgeInsetRect]);
```
输出
```bash
2015-12-23 18:06:21.499 GestureTest1[9353:146428] Inset rect x: 110, y: 90, w: 80, h: 120
2015-12-23 18:06:21.500 GestureTest1[9353:146428] Offset rect: NSRect: {{120, 80}, {100, 100}}
2015-12-23 18:06:21.500 GestureTest1[9353:146428] Edge inset rect: NSRect: {{90, 90}, {150, 140}}
```