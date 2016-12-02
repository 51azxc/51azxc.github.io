title: "UIPageControl与UIScrollView实现图片循环播放的效果"
date: 2016-01-08 18:33:37
tags: ["scrollview", "pagecontrol"]
categories: ["iOS","Objective-C"]
---

> [UIScrollView,UIPageControl,UIImageView 实现图片轮播的效果](http://www.cnblogs.com/imzzk/p/uiscrollview_uiimageview.html)
 
```objc
 @interface PageViewController ()<UIScrollViewDelegate>

@property (strong, nonatomic) UIScrollView *myScrollView;
@property (strong, nonatomic) UIPageControl *myPageControl;

@end

@implementation PageViewController

@synthesize myScrollView, myPageControl;

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    CGFloat width = CGRectGetWidth([UIScreen mainScreen].applicationFrame);
    CGFloat height = CGRectGetHeight([UIScreen mainScreen].applicationFrame);
    
    myScrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 0, width, height)];
    //滚动自动到边界
    [myScrollView setPagingEnabled:YES];
    //不显示滚动条
    [myScrollView setShowsHorizontalScrollIndicator:NO];
    [myScrollView setShowsVerticalScrollIndicator:NO];
    //设置可滚动的区域
    [myScrollView setContentSize:CGSizeMake(width * 3, height)];
    for (NSInteger i=0; i < 3; i++) {
        //添加子区域
        UIView *view = [[UIView alloc] initWithFrame:CGRectMake(width * i, 0, width, height)];
        //添加随机背景色
        [view setBackgroundColor:[UIColor colorWithRed: arc4random_uniform(255)/255.0 green: arc4random_uniform(255)/255.0 blue: arc4random_uniform(255)/255.0 alpha: 1.0]];
        [myScrollView addSubview:view];
    }
    [myScrollView setDelegate:self];
    [self.view addSubview:myScrollView];
    
    myPageControl = [[UIPageControl alloc] initWithFrame:CGRectMake(0, self.view.frame.size.height - 20, width, 20)];
    //设置背景颜色
    [myPageControl setBackgroundColor:[UIColor clearColor]];
    //设置没有选中的圆点颜色
    [myPageControl setPageIndicatorTintColor:[UIColor grayColor]];
    //设置选中状态的圆点颜色
    [myPageControl setCurrentPageIndicatorTintColor:[UIColor whiteColor]];
    //当前页面
    [myPageControl setCurrentPage:0];
    //总页数
    [myPageControl setNumberOfPages:3];
    //添加页面滚动时触发的方法
    [myPageControl addTarget:self action:@selector(pageScroll) forControlEvents:UIControlEventValueChanged];
    [self.view addSubview:myPageControl];
    //自动轮换页面
    [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(loadPage) userInfo:nil repeats:YES];
}

- (void)pageScroll {
    NSInteger page = myPageControl.currentPage;
    CGRect rect = CGRectOffset(myScrollView.frame, myScrollView.frame.size.width * page, 0);
    [myScrollView scrollRectToVisible:rect animated:YES];
}
//scrollView的代理方法，当scrollView减速滚动时执行，更改pageControl的当前页面计数
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView {
    NSInteger page = floor(myScrollView.contentOffset.x / myScrollView.frame.size.width);
    [myPageControl setCurrentPage:page];
}

- (void)loadPage {
    if (myPageControl.currentPage == myPageControl.numberOfPages - 1) {
        [myScrollView setContentOffset:CGPointZero animated:YES];
        myPageControl.currentPage = 0;
    }else {
        myPageControl.currentPage++;
        CGPoint point = CGPointMake(CGRectGetWidth(myScrollView.frame) * myPageControl.currentPage, 0);
        [myScrollView setContentOffset:point animated:YES];
    }
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```