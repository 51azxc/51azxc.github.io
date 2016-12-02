title: "iOS手势练习"
date: 2015-12-23 17:35:43
tags: "gesture"
categories: ["iOS","Objective-C"]
---

> [iphone练习之手势识别(双击、捏、旋转、拖动、划动、长按）UITapGestureRecognizer](http://blog.csdn.net/chang6520/article/details/7924313)

```objc
@interface ViewController ()

@property (strong, nonatomic) UILabel *label;

@end

@implementation ViewController

@synthesize label;

- (void)viewDidLoad {
    [super viewDidLoad];
    
    label = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 150, 60)];
    label.center = self.view.center;
    label.text = @"Test";
    label.textAlignment = NSTextAlignmentCenter;
    [self.view addSubview:label];
    
    //设置点击手势
    UITapGestureRecognizer *tapGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleTapGesture:)];
    //可以设置点击次数
    //tapGesture.numberOfTapsRequired = 2;
    [self.view addGestureRecognizer:tapGesture];
    
    //捏手势
    UIPinchGestureRecognizer *pinchGesture = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(handlePinchGesture:)];
    [self.view addGestureRecognizer:pinchGesture];
    
    //旋转手势
    UIRotationGestureRecognizer *rotationGesture = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(handleRotationGesture:)];
    [self.view addGestureRecognizer:rotationGesture];
    
    //划动手势
    UISwipeGestureRecognizer *swipeGesture = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(handleSwipeGesture:)];
    //向左划，默认为右边
    //swipeGesture.direction = UISwipeGestureRecognizerDirectionLeft;
    [self.view addGestureRecognizer:swipeGesture];
    
    //拖动手势
    UIPanGestureRecognizer *panGesture = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handlePanGesture:)];
    [self.view addGestureRecognizer:panGesture];
    
    UILongPressGestureRecognizer *longPressGesture = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(handleLongPressGesture:)];
    [self.view addGestureRecognizer:longPressGesture];

}

- (void)handleTapGesture: (UIGestureRecognizer *)sender {
    label.text = @"Tap";
}

- (void)handlePinchGesture: (UIGestureRecognizer *)sender {
    
    CGFloat scale = [(UIPinchGestureRecognizer *)sender scale];
    label.text = [@"Pinch" stringByAppendingFormat:@" %f", scale];
    if (scale) {
        label.transform = CGAffineTransformMakeScale(scale, scale);
    }else {
        label.transform = CGAffineTransformMakeScale(scale, scale);
    }
}

- (void)handleRotationGesture: (UIGestureRecognizer *)sender {
    CGFloat rotation = [(UIRotationGestureRecognizer *)sender rotation];
    label.text = [@"rotation" stringByAppendingFormat:@" %f", rotation];
    label.transform = CGAffineTransformMakeRotation(rotation);
}

- (void)handleSwipeGesture: (UIGestureRecognizer *)sender {
    UISwipeGestureRecognizerDirection direction = [(UISwipeGestureRecognizer *)sender direction];
    switch (direction) {
        case UISwipeGestureRecognizerDirectionDown:
            label.text = @"Swipe Down";
            break;
        case UISwipeGestureRecognizerDirectionLeft:
            label.text = @"Swipe Left";
            break;
        case UISwipeGestureRecognizerDirectionRight:
            label.text = @"Swipe Right";
            break;
        case UISwipeGestureRecognizerDirectionUp:
            label.text = @"Swipe Up";
            break;
        default:
            break;
    }
}

- (void)handlePanGesture: (UIGestureRecognizer *)sender {
    CGPoint point = [(UIPanGestureRecognizer *)sender translationInView:self.view];
    label.text = [@"Pan" stringByAppendingFormat:@" %@", [NSValue valueWithCGPoint:point]];
    label.transform = CGAffineTransformMakeTranslation(point.x, point.y);
}

- (void)handleLongPressGesture: (UIGestureRecognizer *)sender {
    label.text = @"Long Press";
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end

```