title: "iOS tableview"
date: 2015-04-09 16:32:26
tags: "TableView"
categories: ["iOS","Objective-C"]
---

> [代码实现 UITableView与UITableViewCell](http://blog.csdn.net/duxinfeng2010/article/details/7724628)
> [UITableView使用指南1](http://blog.csdn.net/y041039/article/details/7351982)

----

首先需要给ViewController添加如下两个协议

```objc
@interface ViewController : UIViewController<UITableViewDelegate,UITableViewDataSource>
```
基本实现如下

```objc
#import "ViewController.h"

@interface ViewController ()
{
    NSArray *dataList;
    UITableView *myTableView;
}

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //UITableViewStylePlain为方形
    //UITableViewStyleGrouped为圆角，类似系统设置页面
    myTableView = [[UITableView alloc] initWithFrame:self.view.frame style:UITableViewStylePlain];
    //指定分割线颜色
    myTableView.separatorColor = [UIColor redColor];
    myTableView.delegate = self;
    myTableView.dataSource = self;
    
    dataList = [[NSArray alloc] initWithObjects:@"1",@"2",@"3",@"4", nil];
    
    [self.view addSubview: myTableView];
}
//返回section个数
-(NSInteger) numberOfSectionsInTableView:(UITableView *)tableView
{
    return 1;
}
//返回table行数
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return  [dataList count];
}
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *cellIdentifier = @"Cell";
    //使用定义的标记符表示需要重用的单元
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
    //如果没有多余的单元，则需创建
    if (cell == nil) {
        //UITableViewCellStyleDefault: 不能显示detailLabelText
        //UITableViewCellStyleSubtitle: detailLabelText显示在第二行
        //UITableViewCellStyleValue1: textLabel与detailTextLabel分别显示在两边
        //UITableViewCellStyleValue2: textLabel与detailTextLabel粘在一起显示
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];
    }
    
    cell.detailTextLabel.text = @"detail";
    //显示数组里面的内容
    cell.textLabel.text = [dataList objectAtIndex:indexPath.row];
    
    //UITableViewCellAccessoryNone: cell最右边不显示任何图标
    //UITableViewCellAccessoryCheckmark: 显示勾
    //UITableViewCellAccessoryDetailButton: 显示详细信息图标
    //UITableViewCellAccessoryDetailDisclosureButton: 显示详细信息图标与小箭头
    //UITableViewCellAccessoryDisclosureIndicator: 显示一个小箭头
    //如果不能满足可以使用cell.accessoryView来重绘指定一个UIView
    cell.accessoryType = UITableViewCellAccessoryNone;
    
    //UITableViewCellSelectionStyleNone: cell选中状态为没有反应
    //UITableViewCellSelectionStyleDefault: 默认状态
    //UITableViewCellSelectionStyleBlue: 选中为蓝色状态
    //UITableViewCellSelectionStyleGray: 选中为灰色状态
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
    
    return cell;
}
//设置行高
-(CGFloat) tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    return 60;
}
//行缩进
-(NSInteger)tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath
{
    if ([indexPath row] % 2 == 0) {
        return 0;
    }
    return 1;
}
//选择事件
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSLog(@"choose: %@",[dataList objectAtIndex:[indexPath row]]);
    //获取当前cell在view中的位置
    CGRect cellRect = [self.tableview convertRect: [self.tableView rectForRowAtIndexPath: indexPath] toView: self.view];
    NSLog(@"cell rect: %@", [NSValue valueForRect: cellRect]);
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}


@end
```