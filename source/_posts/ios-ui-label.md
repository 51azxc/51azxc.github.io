title: "UILabel相关知识"
date: 2015-12-25 15:39:33
tags: "label"
categories: ["iOS","Objective-C"]
---

### 给UILabel添加下划线

> [给UILabel 或者 UIButton标题加下划线](http://blog.csdn.net/chaoyuan899/article/details/38306141)
> [iOS add bottom border to UILabel with shadow](http://stackoverflow.com/questions/12329079/ios-add-bottom-border-to-uilabel-with-shadow)

```objc
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 200, 50)];

[label setText:@"test"];
[label setBackgroundColor:[UIColor clearColor]];
[self.view addSubview:lbl];
[label sizeToFit];

CALayer* layer = [label layer];

CALayer *bottomBorder = [CALayer layer];
bottomBorder.borderColor = [UIColor darkGrayColor].CGColor;
bottomBorder.borderWidth = 1;
bottomBorder.frame = CGRectMake(-1, layer.frame.size.height-1, layer.frame.size.width, 1);
[bottomBorder setBorderColor:[UIColor blackColor].CGColor];
[layer addSublayer:bottomBorder];
```

----

### 让UILabel与UITextField支持缩进

> [UILabel text margin](http://stackoverflow.com/questions/3476646/uilabel-text-margin)
> [UITextField align left margin](http://stackoverflow.com/questions/5674655/uitextfield-align-left-margin)

写一个继承于`UILabel`的子类，然后重写`drawTextInRect`方法:
```objc
- (void)drawTextInRect:(CGRect)rect {
    //缩进
    UIEdgeInsets insets = {0, 15, 0, 0};
    [super drawTextInRect:UIEdgeInsetsInsetRect(rect, insets)];
}
```

继承于`UITextField`的子类,然后重写`textRectForBounds`与`editingRectForBounds`方法
```objc
@interface IndentText : UITextField

@end

@implementation IndentText

static CGFloat leftMargin = 15;

- (CGRect)textRectForBounds:(CGRect)bounds{
    bounds.origin.x += leftMargin;
    return bounds;
}

- (CGRect)editingRectForBounds:(CGRect)bounds{
    bounds.origin.x += leftMargin;
    return bounds;
}

@end
```

----

### UILabel根据字体动态调节高度

> [ios7.0 动态获取长度和高度方法](http://www.cocoachina.com/bbs/read.php?tid=243266)
> [TextKit学习（四）通过boundingRectWithSize:options:attributes:context:计算文本尺寸](http://blog.csdn.net/jymn_chen/article/details/10949279)
> [根据字体多少使UILabel自动调节尺寸](http://blog.csdn.net/enuola/article/details/8559588)

原本使用的`sizeWithFont:constrainedToSize:lineBreakMode:`方法已经开始冒黄金感叹号了，为了让警告消失需要使用`boudingRectWithSize:options:attributes:context`方法了

```objc
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 200, 60)];
label.text = @"The NSMutableParagraphStyle class adds methods to its superclass, NSParagraphStyle, for changing the values of the subattributes in a paragraph style attribute. See the NSParagraphStyle and NSAttributedString specifications for more information.";
//label字体大小
label.font = [UIFont systemFontOfSize:16.0f];
//断句模式
label.lineBreakMode = NSLineBreakByCharWrapping;

NSMutableParagraphStyle *style = [NSMutableParagraphStyle new];
style.lineBreakMode = NSLineBreakByCharWrapping;
NSDictionary *attr = @{ NSFontAttributeName:label.font, NSParagraphStyleAttributeName:style.copy };
//size: 文本占据的大小
//options: 文本选项
//attributes: 文字属性
//context: 上下文
CGSize labelSize = [label.text boundingRectWithSize:CGSizeMake(label.frame.size.width, 999.0) options:NSStringDrawingUsesLineFragmentOrigin attributes:attr context:nil].size;
NSLog(@"label size: %@", [NSValue valueWithCGSize:labelSize]);

label.frame = CGRectMake(0, 0, labelSize.width, labelSize.height);
label.center = self.view.center;
label.backgroundColor = [UIColor blueColor];
label.textColor = [UIColor grayColor];
//设定label的行数
label.numberOfLines = 20;
[self.view addSubview:label];
```