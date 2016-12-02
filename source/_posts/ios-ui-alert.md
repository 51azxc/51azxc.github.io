title: "iOS 弹出框"
date: 2015-04-09 11:08:37
tags: ["UIView", "UIAlertController"]
categories: ["iOS","Objective-C"]
---

### AlertView

> [UIAlertView的基本用法与UIAlertViewDelegate对对话框的事件处理方法](http://blog.csdn.net/enuola/article/details/7900346)

首先需要实现协议`UIAlertViewDelegate`,

```objc
@interface ViewController:UIViewController<UIAlertViewDelegate>
```

基本用法如下

```objc
UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Title" message:@"Hello World" delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"OK", nil];
//查看是否显示
NSLog(@"%d", alert.visible);
//添加按钮
[alert addButtonWithTitle:@"other"];
//按钮总数
NSLog(@"buttons: %d",alert.numberOfButtons);
//根据索引获取按钮标题
NSLog(@"button1: %@",[alert buttonTitleAtIndex:1]);
//获取取消按钮索引
NSLog(@"cancel button index: %d",alert.cancelButtonIndex);
//显示alert
[alert show];

//相关协议方法
//点击按钮触发的事件
-(void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    NSLog(@"click button index: %d",buttonIndex);
    if (buttonIndex == alertView.cancelButtonIndex) {
        NSLog(@"cancel");
    }
}
//显示之前的事件
-(void)willPresentAlertView:(UIAlertView *)alertView
{
    NSLog(@"willPresentAlertView");
}
//显示之后的事件
-(void)didPresentAlertView:(UIAlertView *)alertView
{
    NSLog(@"didPresentAlertView");
}
//消失之前的事件
-(void)alertView:(UIAlertView *)alertView willDismissWithButtonIndex:(NSInteger)buttonIndex
{
    NSLog(@"willDismissWithButtonIndex");
}
//消失完成后的事件
-(void)alertView:(UIAlertView *)alertView didDismissWithButtonIndex:(NSInteger)buttonIndex
{
    NSLog(@"didDismissWithButtonIndex");
}
//被系统退出后的事件
-(void)alertViewCancel:(UIAlertView *)alertView
{
    NSLog(@"alertViewCancel");
}
```

----

## alertview点击cancelButton后alertViewCancel不响应

> [- (void)alertViewCancel:(UIAlertView *)alertView is not called](http://stackoverflow.com/questions/2448244/voidalertviewcanceluialertview-alertview-is-not-called)

alertViewCancel方法是系统退出了alertview时被触发，而不是点击了cancelButton后被触发，如果需要指定点击退出按钮触发，可以使用`clickedButtonAtIndex`方法，通过其中的按钮索引判断是否点击了退出按钮再做下一步
```objc
-(void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (buttonIndex == alertView.cancelButtonIndex) {
        
    }
}
```

----

## AlertController

> [UIAlert​Controller](http://nshipster.cn/uialertcontroller/)
> [UIAlertViewController](http://www.zero1993.com/uialertviewcontroller.html)

**iOS8**新增了`UIAlertController`用来代替之前的`UIAlertView`以及`UIActionSheet`。如今需要实现着两种弹出提示框不再需要实现各种的代理`<UIAlertViewDelegate>,<UIActionSheetDelegate>`,只需要指定对应的样式即可

* `UIAlertControllerStyleActionSheet`: 对应之前的`UIActionSheet`
* `UIAlertControllerStyleAlert`: 对应之前的`UIAlertView`

对于新的`UIAlert`来说最大的变化可以添加任意的`TextField`,这样制作一个简单的输入框就方便多了

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {

    UIAlertController *actionSheet = [UIAlertController alertControllerWithTitle:@"UIAlertController" message:@"actionSheet or alert" preferredStyle:UIAlertControllerStyleActionSheet];
    
    UIAlertAction *defaultAlertAction = [UIAlertAction actionWithTitle:@"默认样式" style:UIAlertActionStyleDefault handler:^(UIAlertAction* action) {
        
        UIAlertController *defaultAlert = [UIAlertController alertControllerWithTitle:@"title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
        
        [defaultAlert addAction:[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil]];
        [defaultAlert addAction:[UIAlertAction actionWithTitle:@"Cancel" style:UIAlertActionStyleCancel handler:nil]];
        
        [self presentViewController:defaultAlert animated:YES completion:nil];
    }];
    
    UIAlertAction *multiBtnAlertAction = [UIAlertAction actionWithTitle:@"多按钮" style:UIAlertActionStyleDefault handler:^(UIAlertAction* action) {
        
        UIAlertController *multiBtnAlert = [UIAlertController alertControllerWithTitle:@"title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
        
        //UIAlertActionStyleDefault为默认样式
        [multiBtnAlert addAction:[UIAlertAction actionWithTitle:@"Yes" style:UIAlertActionStyleDefault handler:^(UIAlertAction* action) {
            NSLog(@"Yes");
        }]];
        //UIAlertActionStyleDestructive为警告样式
        [multiBtnAlert addAction:[UIAlertAction actionWithTitle:@"No" style:UIAlertActionStyleDestructive handler:^(UIAlertAction* action) {
            NSLog(@"No");
        }]];
        //UIAlertActionStyleCancel为取消样式
        [multiBtnAlert addAction:[UIAlertAction actionWithTitle:@"Cancel" style:UIAlertActionStyleCancel handler:^(UIAlertAction* action) {
            NSLog(@"Cancel");
        }]];
        
        [self presentViewController:multiBtnAlert animated:YES completion:nil];
    }];
    
    UIAlertAction *inputAlertAction = [UIAlertAction actionWithTitle:@"带输入框" style:UIAlertActionStyleDefault handler:^(UIAlertAction* action) {
        
        UIAlertController *inputAlert = [UIAlertController alertControllerWithTitle:@"title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];
        
        [inputAlert addTextFieldWithConfigurationHandler:^(UITextField*  textField) {
            textField.placeholder = @"username";
        }];
        
        [inputAlert addTextFieldWithConfigurationHandler:^(UITextField* textField) {
            textField.placeholder = @"password";
            textField.secureTextEntry = YES;
        }];
        
        [inputAlert addAction:[UIAlertAction actionWithTitle:@"Sign In" style:UIAlertActionStyleDefault handler:^(UIAlertAction* action) {
            UITextField *userInput = inputAlert.textFields.firstObject;
            UITextField *passwdInput = inputAlert.textFields.lastObject;
            NSLog(@"username: %@, password: %@", userInput.text, passwdInput.text);
        }]];
        
        [self presentViewController:inputAlert animated:YES completion:nil];
    }];
    
    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"Cancel" style:UIAlertActionStyleCancel handler:^(UIAlertAction* action) {
        NSLog(@"ActionSheet Cancel");
    }];
    
    [actionSheet addAction:defaultAlertAction];
    [actionSheet addAction:multiBtnAlertAction];
    [actionSheet addAction:inputAlertAction];
    [actionSheet addAction:cancelAction];
    
    [self presentViewController:actionSheet animated:YES completion:nil];
    
}
```