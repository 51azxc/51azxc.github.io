title: "NavigationController相关使用"
date: 2015-04-13 16:56:01
tags: "navigationController"
categories: ["iOS","Objective-C"]
---

### 基本使用

> [iOS学习之UINavigationController详解与使用(一)添加UIBarButtonItem](http://blog.csdn.net/totogo2010/article/details/7681879)

首先修改`appDelegate`的`didFinishLaunchingWithOptions`方法
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds] ];
    //指定主页面
    MainViewController *view = [[MainViewController alloc] init];
    self.window.rootViewController = [[UINavigationController alloc] initWithRootViewController:view];
    [self.window makeKeyAndVisible];
    return YES;
}
```

`MainViewControoler`的初始化方法如下
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    //定义左边按钮
    UIBarButtonItem *leftBtn = [[UIBarButtonItem alloc] initWithTitle:@"Back" style:UIBarButtonItemStyleDone target:self action:@selector(back)];
    self.navigationItem.leftBarButtonItem = leftBtn;
    //定义右边按钮
    UIBarButtonItem *rightBtn = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemAction target:self action:@selector(next)];
    self.navigationItem.rightBarButtonItem = rightBtn;
    //定义当前页面导航条名称
    self.navigationItem.title = @"Test";
    //显示Toolbar
    [self.navigationController setToolbarHidden:NO animated:YES];
}
```
使用`navigationController`的情况下，用`pushViewController`方法将跳转到下一个页面，使用`popViewControllerAnimated`方法将返回到上一个页面

----

### `presentModalViewController`与`pushViewController`共存

> [先presentModalViewController后pushViewController没有效果的解决方法](http://blog.csdn.net/ck89757/article/details/27496453)

在`presentModalViewController`的`ViewController`先用`NatigationController`的初始化方法实例化，在下一层页面就可以使用`pushViewController`的方法了
```objc
ViewController *view = [[ViewController alloc] init];
UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:view];
[self.navigationController presentViewController:nav animated:YES];
```

----

### `pushViewController`到指定View

> [navigationController pushViewController 多次跳转后怎么返回](http://my.oschina.net/wycdavid/blog/196278)

跳转到跟页面
```objc
[self.navigationController popToRootViewController];
```

通过index定位跳转到指定页面
```objc
[self.navigationController popToViewController:[self.navigationController.viewControllers objectAtIndex:2] animated:YES];
```

通过class定位
```objc
for (UIViewController *controller in self.navigationController.viewControllers) {
    if ([controller isKindOfClass:[需要跳转到的Controller class]]) {
        [self.navigationController popToViewController:controller animated:YES];
    }
}
```

----

### 页面组件自适应`NavigationBar`

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

### presentViewController背景半透明

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