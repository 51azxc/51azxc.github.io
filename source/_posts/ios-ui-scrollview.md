title: "UIScrollView使用"
date: 2016-01-08 18:28:40
tags: "scrollview"
categories: ["iOS","Objective-C"]
---

> [第二、UIScrollView的使用大全](http://blog.csdn.net/ch_soft/article/details/6947695)

```objc
@interface ScrollViewController ()<UIScrollViewDelegate>

@property (strong, nonatomic) UIImageView *imgView;

@end

@implementation ScrollViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    self.imgView = [[UIImageView alloc] initWithImage:[UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"test" ofType:@"jpg" ]]];
    //计算最小缩放率
    CGFloat minZoom = MIN(self.view.bounds.size.width / self.imgView.frame.size.width, self.view.bounds.size.height / self.imgView.frame.size.height);
    minZoom = minZoom>1 ? 1:minZoom;
    
    UIScrollView *myScrollView = [[UIScrollView alloc] initWithFrame:self.view.frame];
    //内容尺寸
    myScrollView.contentSize = self.imgView.frame.size;
    //当前缩放倍率
    myScrollView.zoomScale = minZoom;
    //最小缩放倍率
    myScrollView.minimumZoomScale = minZoom;
    //最大缩放倍率
    myScrollView.maximumZoomScale = 3.0;
    //自动滚动到边界,默认值NO
    myScrollView.pagingEnabled = YES;
    //允许滚动
    myScrollView.scrollEnabled = YES;
    //滚动至边界的反弹效果,默认值YES
    myScrollView.bounces = YES;
    //放大至最大倍率的反弹效果
    myScrollView.bouncesZoom = YES;
    //是否显示垂直滚动条
    myScrollView.showsVerticalScrollIndicator = NO;
    //是否显示水平滚动条
    myScrollView.showsHorizontalScrollIndicator = NO;
    //锁定可以垂直水平方向同时运动，如果为YES, 则指允许一开始运动的方向。默认值NO
    myScrollView.directionalLockEnabled = NO;
    //手指放开后的减速率
    myScrollView.decelerationRate = 0.5;
    myScrollView.delegate = self;
    
    [myScrollView addSubview:self.imgView];
    [self.view addSubview:myScrollView];
}

//滚动时触发
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    //NSLog(@"scroll point: %@",[NSValue valueWithCGPoint:scrollView.contentOffset]);
}
//开始拖拽时触发
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView {
    //tracking属性为是否触摸但不拖动
    NSLog(@"tracking: %@", scrollView.tracking ? @"YES":@"NO");
}
//减速停止时触发
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView {
    //decelerating属性为滚动时手指放开是否还在滚动状态
    NSLog(@"decelerating: %@", scrollView.decelerating ? @"YES":@"NO");
}
//放大后的视图
- (UIView *)viewForZoomingInScrollView:(UIScrollView *)scrollView {
    return self.imgView;
}

- (void)scrollViewWillBeginZooming:(UIScrollView *)scrollView withView:(UIView *)view {
    //zooming属性为判断是否正在缩放
    NSLog(@"zooming: %@", scrollView.zooming ? @"YES":@"NO");
}
//放大缩小结束后触发
- (void)scrollViewDidEndZooming:(UIScrollView *)scrollView withView:(UIView *)view atScale:(CGFloat)scale {
    //zoomBouncing属性为判断是否缩放到最大/小值
    NSLog(@"zoomBouncing: %@", scrollView.zoomBouncing ? @"YES":@"NO");
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```