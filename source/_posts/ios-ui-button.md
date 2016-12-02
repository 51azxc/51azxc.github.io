title: "UIButton相关知识"
date: 2015-12-25 15:38:15
tags: "button"
categories: ["iOS","Objective-C"]
---

### UIButton setImage与setBackgroundImage

> [关于 setBackgroundImage 和 setImage](http://blog.csdn.net/reylen/article/details/8504015)

UIButton中如果想要给Button添加图片，可以使用`setImage`方法，使用这种方法，图片不会拉伸，如果使用`setBackgroundImage`方法则会将突破拉伸到覆盖整个button.

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

### UIButton图标与文字上下排列

> [UIButton: how to center an image and a text using imageEdgeInsets and titleEdgeInsets?](http://stackoverflow.com/questions/2451223/uibutton-how-to-center-an-image-and-a-text-using-imageedgeinsets-and-titleedgei)

`UIButton`中加入了图片与文字是居中并排的，如果想要上下排列，可以使用`UIEdgeInsetsMake(top, left, bottom, right)`来实现
```objc
CGSize imageSize = button.imageView.frame.size;
button.titleEdgeInsets = UIEdgeInsetsMake(0.0, -imageSize.width, -imageSize.height, 0.0);
CGSize titleSize = button.titleLabel.frame.size;
button.imageEdgeInsets = UIEdgeInsetsMake(-titleSize.height, 0.0, 0.0, -titleSize.width);
```