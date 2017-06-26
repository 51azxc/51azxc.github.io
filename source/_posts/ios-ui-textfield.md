title: "iOS UITextField相关知识"
date: 2015-04-10 10:26:31
tags: "TextField"
categories: ["iOS","Objective-C"]
---

### UITextField,UIButton基本用法

给`viewController`添加了`UITextFieldDelegate`协议
```objc
//去掉navigation导航条
[self.navigationController setNavigationBarHidden:YES];

// 设置背景颜色
self.view.backgroundColor = [UIColor colorWithRed:(200.0/255.0) green:(200.0/255.0) blue:(200.0/255.0) alpha: 0.8];
//设置字体大小
UIFont *font = [UIFont fontWithName:@"Arial" size:30];
UILabel *label = [[UILabel alloc] initWithFrame: CGRectMake(30, 110, 100, 60)];
label.text = @"请登录";
label.font = font;
[self.view addSubview: label];

UIView *usrView = [[UIView alloc] initWithFrame: CGRectMake(30, 162, 260, 40)];
usrView.backgroundColor = [UIColor whiteColor];
//设置左上/右上角为圆角
UIBezierPath *usrPath = [UIBezierPath bezierPathWithRoundedRect:usrView.bounds byRoundingCorners:UIRectCornerTopLeft|UIRectCornerTopRight cornerRadii:CGSizeMake(5.0f, 5.0f)];
CAShapeLayer *usrLayer = [[CAShapeLayer alloc] init];
usrLayer.frame = usrView.bounds;
usrLayer.path = usrPath.CGPath;
usrView.layer.mask = usrLayer;

//第一个文本框 x,y width,height
usrText = [[UITextField alloc] initWithFrame: CGRectMake(5, 5, 250, 30)];
usrText.backgroundColor = [UIColor whiteColor];
//边框样式
usrText.borderStyle = UITextBorderStyleNone;
//设置委托对象
usrText.delegate = self;
//文字对齐方式
usrText.contentHorizontalAlignment = UIControlContentHorizontalAlignmentLeft;
usrText.contentVerticalAlignment = UIControlContentVerticalAlignmentCenter;
//占位符
usrText.placeholder = @"用户";
//替换弹出键盘的返回键
usrText.returnKeyType = UIReturnKeyNext;
//编辑时出现清除按钮
usrText.clearButtonMode = UITextFieldViewModeWhileEditing;
//将文本框加入到主视图中
[usrView addSubview: usrText];
[self.view addSubview: usrView];

UIView *pwdView = [[UIView alloc] initWithFrame: CGRectMake(30, 204, 260, 40)];
pwdView.backgroundColor = [UIColor whiteColor];
//设置左下/右下角为圆角
UIBezierPath *pwdPath = [UIBezierPath bezierPathWithRoundedRect:pwdView.bounds byRoundingCorners:UIRectCornerBottomLeft|UIRectCornerBottomRight cornerRadii:CGSizeMake(5.0f, 5.0f)];
CAShapeLayer *pwdLayer = [[CAShapeLayer alloc] init];
pwdLayer.frame = pwdView.bounds;
pwdLayer.path = pwdPath.CGPath;
pwdView.layer.mask = pwdLayer;

pwdText = [[UITextField alloc] initWithFrame: CGRectMake(5, 5, 250, 30)];
pwdText.backgroundColor = [UIColor whiteColor];
pwdText.borderStyle = UITextBorderStyleNone;
pwdText.delegate = self;
pwdText.contentHorizontalAlignment = UIControlContentHorizontalAlignmentLeft;
pwdText.contentVerticalAlignment = UIControlContentVerticalAlignmentCenter;
pwdText.placeholder = @"密码";
//加密输入字符
pwdText.secureTextEntry = YES;
pwdText.returnKeyType = UIReturnKeyGo;
pwdText.clearButtonMode = UITextFieldViewModeWhileEditing;
[pwdView addSubview: pwdText];
[self.view addSubview: pwdView];

loginBtn = [[UIButton alloc] initWithFrame:CGRectMake(30, 264, 260, 40)];
//按钮颜色
loginBtn.backgroundColor = [UIColor colorWithRed: (48.0/255.0) green: (113.0/255.0) blue: (169.0/255.0) alpha:1.0];
//圆角半径
[loginBtn.layer setCornerRadius:5.0];
//边框宽度
[loginBtn.layer setBorderWidth:1.0];
//文字
[loginBtn setTitle:@"登录" forState:UIControlStateNormal];
//文字颜色
[loginBtn setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
loginBtn.tintColor = [UIColor whiteColor];
//添加点击事件
[loginBtn addTarget: self action: @selector(loginBtnClick:) forControlEvents:UIControlEventTouchUpInside];
[self.view addSubview: loginBtn];

```

----

### UITextField 基本知识

> [iOS SDK基础知识:UITextView & UITextViewDelegate](http://www.cocoachina.com/industry/20130419/6037.html)
> [IOS-TextField知多少](http://blog.sina.com.cn/s/blog_6ec3c9ce01015599.html)
> [IOS键盘的相关设置(UITextfield)](http://blog.csdn.net/h3c4lenovo/article/details/8447661)
> [UITextField设置placeholder颜色](http://code4app.com/snippets/one/UITextField%E8%AE%BE%E7%BD%AEplaceholder%E9%A2%9C%E8%89%B2/537c765e933bf0a0468b525c)

#### 键盘类型
```objc
usrText.keyboardtype = UIKeyboardTypeDefault;
typedef enum {  
    UIKeyboardTypeDefault,                // 默认键盘：支持所有字符   
    UIKeyboardTypeASCIICapable,           // 支持ASCII的默认键盘   
    UIKeyboardTypeNumbersAndPunctuation,  // 标准电话键盘，支持+*#等符号   
    UIKeyboardTypeURL,                    // URL键盘，有.com按钮；只支持URL字符   
    UIKeyboardTypeNumberPad,              //数字键盘   
    UIKeyboardTypePhonePad,               // 电话键盘   
    UIKeyboardTypeNamePhonePad,           // 电话键盘，也支持输入人名字   
    UIKeyboardTypeEmailAddress,           // 用于输入电子邮件地址的键盘   
} UIKeyboardType;  

```

#### 返回键
```objc
usrText.returnKeyType = UIReturnKeyNext;
typedef enum {  
    UIReturnKeyDefault, //默认：灰色按钮，标有Return
    UIReturnKeyGo,  //标有Go的蓝色按钮
    UIReturnKeyGoogle, //标有Google的蓝色按钮，用于搜索
    UIReturnKeyJoin, //标有Join的蓝色按钮
    UIReturnKeyNext, //标有Next的蓝色按钮
    UIReturnKeyRoute, //标有Route的蓝色按钮
    UIReturnKeySearch, //标有Search的蓝色按钮
    UIReturnKeySend, //标有Send的蓝色按钮
    UIReturnKeyYahoo, //标有Yahoo!的蓝色按钮，用于搜索
    UIReturnKeyDone, //标有Done的蓝色按钮
    UIReturnKeyEmergencyCall, //紧急呼叫按钮
} UIReturnKeyType; 
```

#### 自动大写
```objc
usrText.autocapitalizationType = UITextAutocapitalizationTypeWord;
typedef enum {  
    UITextAutocapitalizationTypeNone, //不自动大写   
    UITextAutocapitalizationTypeWords, //单词首字母大写   
    UITextAutocapitalizationTypeSentences, //句子首字母大写   
    UITextAutocapitalizationTypeAllCharacters, //所有字母大写   
} UITextAutocapitalizationType;
```

#### 自动更正
```objc
usrText.autocorrectionType = UITextAutocorrectionTypeYes;
typedef enum {  
    UITextAutocorrectionTypeDefault,//默认   
    UITextAutocorrectionTypeNo,//不自动更正   
    UITextAutocorrectionTypeYes,//自动更正   
} UITextAutocorrectionType;  
```

#### 边框
```objc
usrText.borderStyle = UITextBorderStyleNone;
typedef enum {
    UITextBorderStyleNone, 
    UITextBorderStyleLine,
    UITextBorderStyleBezel,
    UITextBorderStyleRoundedRect  
} UITextBorderStyle;
```

#### 外观
```objc
usrText.keyboardAppearance=UIKeyboardAppearanceDefault;
typedef enum {  
    UIKeyboardAppearanceDefault,    // 默认外观：浅灰色   
    UIKeyboardAppearanceAlert,      //深灰/石墨色   
} UIKeyboardAppearance;
```

#### 清除键
```objc
usrText.clearButtonMode = UITextFieldViewModeWhileEditing;
typedef enum {
    UITextFieldViewModeNever,  不出现
    UITextFieldViewModeWhileEditing, 编辑时出现
    UITextFieldViewModeUnlessEditing,  除了编辑外都出现
    UITextFieldViewModeAlways   一直出现
} UITextFieldViewMode;
```

#### 其他
```objc
//触发通知
UIKeyboardWillShowNotification   //键盘显示之前发送
UIKeyboardDidShowNotification    //键盘显示之后发送
UIKeyboardWillHideNotification   //键盘隐藏之前发送
UIKeyboardDidHideNotification    //键盘隐藏之后发送

//获取焦点(键盘弹出)
[usrText becomeFirstResponder];
//移除焦点(键盘回收)
[usrText resignFirstResponder];

//统计字数
- (void)textViewDidChange:(UITextView *)textView
{
    int count = [textView.text length];
}
//捕获按下回车键事件
- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text
{
    if (1 == range.length) {//回车键
        return YES;
    }
    if ([text isEqualToString:@"\n"]) {//按下return键
        //这里隐藏键盘，不做任何处理
        [textView resignFirstResponder];
        return NO;
    }else {
        if ([textView.text length] < 140) {//判断字符个数
            return YES;
        }  
    }
    return NO;
}
```

----

### 键盘弹出挡住的解决方案

> [iOS学习笔记——视图上移与键盘弹回](http://www.2cto.com/kf/201401/270097.html)
> [IOS 点击空白处隐藏键盘的几种方法](http://blog.csdn.net/swingpyzf/article/details/17091567)
> [iOS点击键盘以外空白区域隐藏键盘的常见方法](http://www.cnblogs.com/jaenson/archive/2013/03/12/iOS_Keybroad_Hide.html)

```objc
//点击空白处隐藏键盘
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    [usrText resignFirstResponder];
    [pwdText resignFirstResponder];
}

//开始编辑时调用方法 -来自UITextFieldDelegate
-(void) textFieldDidBeginEditing:(UITextField *)textField
{
    [self animateTextField:textField up:YES];
}
//编辑结束后调用方法 -来自UITextFieldDelegate
-(void) textFieldDidEndEditing:(UITextField *)textField
{
    [self animateTextField: textField up: NO];
}

//弹出键盘后视图上移方法
-(void) animateTextField: (UITextField *) textField up: (BOOL) up
{
    //设定上移偏移量，单位像素
    const int move = 80;
    //判定是否要上移
    int movement = (up ? -move:move);
    //设置动画名字
    [UIView beginAnimations:@"Animation" context:nil];
    //设置动画开始移动位置
    [UIView setAnimationBeginsFromCurrentState:YES];
    //设置动画持续时间
    [UIView setAnimationDuration: 0.3f];
    //设置动画的移动位移
    self.view.frame = CGRectOffset(self.view.frame, 0, movement);
    //结束动画
    [UIView commitAnimations];
}
```

----

### 设置占位符颜色

> [IOS学习笔记36—解决键盘遮挡输入框（UITextField）问题](http://blog.csdn.net/ryantang03/article/details/8203605)

```objc
textField.attributedPlaceholder = [[NSAttributedString alloc] initWithString:@"密码" attributes:@{NSForegroundColorAttributeName: [UIColor blackColor]}];
```

----

### 默认选中字符

> [Can I select a specific block of text in a UITextField?](http://stackoverflow.com/questions/3277538/can-i-select-a-specific-block-of-text-in-a-uitextfield)

当需要`TextField`默认选中所有字符的时候，可以调用其`selectAll`方法，如果需要选择部分字符，则需使用`setSelectedTextRange`方法:
```objc
- (void)textFieldDidBeginEditing:(UITextField *)textField {
    if ([textField.text containsString:@"."]) {
        //如果为文件名时需要选中文件名部分，无需选中后缀
        NSRange range = [self.name rangeOfString:@"." options:NSBackwardsSearch];
        UITextPosition *beginPosition = [textField positionFromPosition:textField.beginningOfDocument offset:0];
        UITextPosition *endPosition = [textField positionFromPosition:textField.beginningOfDocument offset:range.location];
        UITextRange *newRange = [textField textRangeFromPosition:beginPosition toPosition:endPosition];
        //定义选中区间
        [textField setSelectedTextRange:newRange];
    } else {
        [self.nameTextField selectAll:self];
    }
}
```