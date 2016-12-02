title: "自定义TabBar"
date: 2015-12-24 16:52:35
tags: "TabBar"
categories: ["iOS","Objective-C"]
---

### 自定义TabBar

> [iOS7 Custom UITabBarController](http://blog.csdn.net/u014084081/article/details/21539087)

```objc
FirstViewController *firstViewCtrl = [[FirstViewController alloc] init];
UINavigationController *firstNav = [[UINavigationController alloc] initWithRootViewController:firstViewCtrl];
SecondViewController *secondViewCtrl = [SecondViewController new];
UINavigationController *secondNav = [[UINavigationController alloc] initWithRootViewController:secondViewCtrl];

//加载的视图
self.viewControllers = @[firstNav, secondNav];
//显示tabBar
//self.tabBar.hidden = YES;
//设置选中的图片
[self.tabBar setSelectionIndicatorImage:[UIImage imageNamed: @"selected"]];
//更改第一个按钮的名称
[[self.tabBar.items objectAtIndex:0] setTitle:@"First"];
//设置按钮名称的位置，默认在底部
[[self.tabBar.items objectAtIndex:0] setTitlePositionAdjustment:UIOffsetMake(0, -15)];
//设置按钮名称属性，主要是修改字体大小，默认比较小
[[self.tabBar.items objectAtIndex:0] setTitleTextAttributes:[NSDictionary dictionaryWithObjectsAndKeys:[UIColor whiteColor], NSForegroundColorAttributeName,[UIFont systemFontOfSize:16], NSFontAttributeName, nil] forState:UIControlStateNormal];
//同样设置第二个按钮
[[self.tabBar.items objectAtIndex:1] setTitle:@"Second"];
[[self.tabBar.items objectAtIndex:1] setTitlePositionAdjustment:UIOffsetMake(0, -15)];
[[self.tabBar.items objectAtIndex:1] setTitleTextAttributes:[NSDictionary dictionaryWithObjectsAndKeys:[UIColor whiteColor], NSForegroundColorAttributeName,[UIFont systemFontOfSize:16], NSFontAttributeName, nil] forState:UIControlStateNormal];
//设置tabBar的位置大小，主要更改位置，默认tabBar在底部
self.tabBar.frame = CGRectMake(0, 64, self.tabBar.frame.size.width, self.tabBar.frame.size.height);
//设置tabBar的背景图片
[self.tabBar setBackgroundImage:[UIImage imageNamed: @"un_selected"]];
```