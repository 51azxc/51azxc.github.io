title: "iOS 数学函数"
date: 2015-05-26 12:06:25
tags: "math"
categories: ["iOS","Objective-C"]
---

### ios 数学函数
> [objective-ｃ适用c数学函数 `<math.h>`](http://www.cnblogs.com/pengyingh/articles/2451278.html)

函数在`<math.h>`中定义
数学常量：
```objc
#define M_E         2.71828182845904523536028747135266250   // e
#define M_LOG2E     1.44269504088896340735992468100189214   // log 2e
#define M_LOG10E    0.434294481903251827651128918916605082  // log 10e
#define M_LN2       0.693147180559945309417232121458176568  // log e2
#define M_LN10      2.30258509299404568401799145468436421   // log e10
#define M_PI        3.14159265358979323846264338327950288   // pi
#define M_PI_2      1.57079632679489661923132169163975144   // pi/2
#define M_PI_4      0.785398163397448309615660845819875721  // pi/4
#define M_1_PI      0.318309886183790671537767526745028724  // 1/pi
#define M_2_PI      0.636619772367581343075535053490057448  // 2/pi
#define M_2_SQRTPI  1.12837916709551257389615890312154517   // 2/sqrt(pi)
#define M_SQRT2     1.41421356237309504880168872420969808   // sqrt(2)
#define M_SQRT1_2   0.707106781186547524400844362104849039  // 1/sqrt(2)
```

常用函数：
```objc
//指数运算
NSLog(@"%.f", pow(3,2) ); //result 9
NSLog(@"%.f", pow(3,3) ); //result 27

//开平方运算
NSLog(@"%.f", sqrt(16) ); //result 4
NSLog(@"%.f", sqrt(81) ); //result 9

//上舍入
NSLog(@"res: %.f", ceil(3.000000000001)); //result 4
NSLog(@"res: %.f", ceil(3.00)); //result 3

//下舍入
NSLog(@"res: %.f", floor(3.000000000001)); //result 3
NSLog(@"res: %.f", floor(3.9999999)); //result 3

//四舍五入
NSLog(@"res: %.f", round(3.5)); //result 4
NSLog(@"res: %.f", round(3.46)); //result 3
NSLog(@"res: %.f", round(-3.5)); //NB: this one returns -4

//最小值
NSLog(@"res: %.f", fmin(5,10)); //result 5

//最大值
NSLog(@"res: %.f", fmax(5,10)); //result 10

//绝对值
NSLog(@"res: %.f", fabs(10)); //result 10
NSLog(@"res: %.f", fabs(-10)); //result 10

```
