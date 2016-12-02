title: "iOS 回调方式"
date: 2015-05-08 16:52:26
tags: ["block", "callback", "performSelector", "respondsToSelector"]
categories: ["iOS","Objective-C"]
---

### IOS 回调方式
> [细数Objective-C中的回调机制](http://blog.sina.com.cn/s/blog_631af5500100z4ub.html)

#### 协议
协议主要是提供接口、或是类似C++多重继承功能，为类提供一种修饰机制。协议不是为回调而生的，它应该表述一组互操作约定。
**优点**：
1. 实现简单，容易理解。
2. 强类型检查。
**缺点**：
1. 类与类间建立了比较强的耦合关系
2. 有可能需要较长期保存委托以进行回调。如果保留的委托需要有独占性，可能会给单件模式、以及多线程带来麻烦。
3. 类只能通过一个方法完成一种类型的回调。代码逻辑很容易集中到一个方法中。
4. 大部分回调使用实际无需通过协议暴露给外部。

----------

#### 函数指针
传统的C语言回调机制。
**优点**：
1. 轻量级的回调机制。
2. 只约定返回值和参数，而非函数名。无参数、返回值限制，使用灵活。
3. 编译器提供类型检查。（错误时产生警告）

**缺点**：
1. 与OBJC的消息机制不兼容。因为消息并非C语言中那样，函数名对应函数指针。即只能对C函数进行回调。
2. 传入不符合约定的函数指针时，产生副作用继续运行，而非报错。

----------

#### objc_msgSend
通过导入`#import <objc/message.h>`获得运行时的消息调用。
其定义为
```objc
id objc_msgSend(id theReceiver, SEL theSelector, ...)
```
**优点**：
1. 轻量级的回调机制。
2. 无传入参数限制。
3. 相比`performSelector`，使用自动引数特征时，不产生警告。
4. 同系列的方法支持`double`、`struct`等类型的返回值，但仍然不支持`int`型返回值（可使用`NSNumber`包装以回避）。

**缺点**：传入不符合约定的消息时，产生副作用继续运行，而非报错。

----------

#### IMP
IMP类似于OBJC提供的函数指针，它通过`methodForSelector`方法查询传入的`Selector`，以获得函数的入口地址。

其定义为
```objc
id (*IMP)(id, SEL, ...)
```
相比普通C语言的函数指针，其定义多了id,SEL这两个强制参数约定，其他与函数指针无异。

**优点**：
1. 轻量级的回调机制。
2. 传入不符合约定的消息时，报错。
3. 无传入参数限制。返回值可通过强转获得，无类型限制。如：
```objc
typedef int (*CBFUNC)(id, SEL, int, int, int); // 定义函数指针类型
int ret = ((CBFUNC)callback)(self, sel, param1, param2, param3); // 强制转换
```
这里的id和SEL只是OBJC系统约定的占位，自定义回调时无实际意义。
由于此阶段实际是函数指针调用，因此最好还是typedef定义函数指针，然后对IMP强转一下，以免出现错误，也能提供一些编译期保护。

**缺点**：依然不能提供如同协议和函数指针的编译期类型检查

----------

#### 使用`respondsToSelector`和`performSelector`进行回调。
利用OBJC的运行时特性，查找对象的消息进行回调

**优点**：
1. 与OBJC代码兼容性好。
2. 具有延迟执行等特性。
3. 轻量级的回调机制。

缺点：
1. 回调产生的返回值只能为id类型，int等类型会产生错误。
2. 参数最多只能传入两个。但可以通过建立包含多个参数的参数类进行回避。同时返回值限制也可通过此方式解决，即建立一个输入类和一个输出类。`NSInvocation`也提供了多参数的解决方法。
3. 如果以`[target performSelector: @selector(callback)];`方式建立回调，则需要对类的回调消息名建立约定，且回调消息名具有独占性，即一个类中只能以此消息名进行回调。
如果通过外部传入SEL建立回调`[target performSelector: sel];`或是外部传入字符串建立回调`[target performSelector:NSSelectorFromString(@"callback")];`
使用自动引数编译器特征（ARC）会产生警告“performSelector may cause a leak because its selector is unknown”
使用此种方式建立回调，当传入一个不符合约定的消息时，会产生副作用继续运行，而非报错。比如约定消息有2个参数，但传入消息只有1个参数，则按照参数约定顺序屏蔽掉最后传入的参数。或是传入消息具有3个参数，则多余的参数值未初始化。

----------

#### NSNotificationCenter
`NSNotificationCenter`是OBJC提供的消息机制。它有些类似于观察者模式，通过关注感兴趣的消息，建立回调。`NSNotificationCenter`提供了一种低耦合的对象通讯机制，特别适合无指定对象的一对多回调。

主要方法：

1）获取消息中心实例（系统已创建，单件模式）
```objc
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
```
2）发送消息。(事件发生时调用)
```objc
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
[nc postNotificationName: NOTIFY_MSG_UC_COMMON_PLAYER_PLAY   // 消息名（字符串）
    object:self       // 消息源
    userInfo:nil];    // 用户字典（传递更多自定义参数）
```
3）注册消息
```objc
[nc addObserver: self                              // 观察者
    selector: @selector(handleNotify_Play:)        // 回调
    name: NOTIFY_MSG_UC_COMMON_PLAYER_PLAY         // 监听消息
    object: nil];                                  // 消息源
```
4）注销消息
```objc
[nc removeObserver: self];
```
5）回调定义
```objc
- (void) handleNotify_Play:(NSNotification *)note;
```
   只有一个参数
```objc
NSNotification*
 –name      // 消息名
 –object    // 消息源
 –userInfo  // 用户字典
```

**优点**：
1. 回调对象间耦合度低。相互之间可不必知道对方存在。
2. 通过消息传递的信息无限制。
3. 观察者可选择特定消息、特定对象，或者特定对象的特定消息进行观察。

**缺点**：缺乏时序性。当事件发生时，回调执行的先后次序不确定。也不能等待回调完成执行后续操作。解决：1）使用传统回调机制。2）多线程时，可使用NSCondition同步线程。3）使用更多的消息。（过多使用可能导致混乱）


----------

#### Block
Block是OBJC提供的一种运行时方法机制，类似于`Javascript`的匿名函数。它提供了一种运行时的临时回调机制。
**注意**：
1）block对象使用的变量、参数在运行时被绑定，因此可以直接使用栈空间建立的变量，无需参数传入。但block对象的创建依然有生命周期限制，因此传入异步调用的block对象时，如果是栈空间创建的block，必须使用`Block_copy()`将block拷出备份，然后使用`Block_release()`将block释放
2）对于在栈空间声明的变量，绑定到block时被标记为const。只能读取不能写入。如果需要写入，需要用__block对变量进行标记。此时block使用的是从栈拷贝到堆中的对象。当出block时，如果栈可用则将堆中对象自动拷贝回栈。
**优点**：
1. 最轻量级的回调机制。
2. 编译器类型检查。
3. 如函数指针一样，灵活定义回调函数。
**缺点**：
1. 执行效率。（影响程度不清楚）
2. 容易导致代码逻辑集中。
3. IOS4之后的特性

**总结**：
OBJC还没有太完美的轻量级回调机制，只能根据情况选择合适的机制。
1. 单纯的回调，且没有复用的必要，也无IOS版本限制，可采用`block`。
2. 单纯的回调，有复用要求，可使用`performSelector`、`objc_msgSend`，或是`IMP`的回调机制。
3. 使用自动引数的情况下，尽量不使用performSelector回调传入的`@Selector`，防止警告。
4. 对象间有较多的互操作，对象有复用的必要，可采用协议。
5.  无指定对象的一对多回调采用`NSNotificationCenter`。
6.  有延迟调用等特殊应用的，可以使用`performSelector`。

----------

###  关于performSelector及respondsToSelector的相关使用

> [关于performSelector调用和直接调用区别](http://blog.csdn.net/abel_tu/article/details/12422743)
> [respondsToSelector的相关使用](http://blog.csdn.net/l_ch_g/article/details/8893629)

```objc
[delegate imageDownloader:self didFinishWithImage:image];
[delegate performSelector:@selector(imageDownloader:didFinishWithImage:) withObject:self withObject:image];
```

1、`performSelector`是运行时系统负责去找方法的，在编译时候不做任何校验；如果直接调用编译是会自动校验。如果`imageDownloader：didFinishWithImage:image：`不存在，那么直接调用 在编译时候就能够发现（借助Xcode可以写完就发现），但是使用`performSelector`的话一定是在运行时候才能发现（此时程序崩溃）；Cocoa支持在运行时向某个类添加方法，即方法编译时不存在，但是运行时候存在，这时候必然需要使用`performSelector`去调用。所以有时候如果使用了`performSelector`，为了程序的健壮性，会使用检查方法
```objc
- (BOOL)respondsToSelector:(SEL)aSelector;
```
2、直接调用方法时候，一定要在头文件中声明该方法的使用，也要将头文件`import`进来。而使用`performSelector`时候， 可以不用import头文件包含方法的对象，直接用`performSelector`调用即可。

`respondsToSelector`的相关使用
`-(BOOL) isKindOfClass: classObj` 用来判断是否是某个类或其子类的实例
`-(BOOL) isMemberOfClass: classObj` 用来判断是否是某个类的实例
`-(BOOL) respondsToSelector: selector` 用来判断是否有以某个名字命名的方法(被封装在一个selector的对象里传递)
`+(BOOL) instancesRespondToSelector: selector` 用来判断实例是否有以某个名字命名的方法. 和上面一个不同之处在于, 前面这个方法可以用在实例和类上，而此方法只能用在类上.
`-(id) performSelector: selector`
```objc
SEL sel = @selector (start:) ; // 指定action  
if ([obj respondsToSelector:sel]) 
{ //判断该对象是否有相应的方法  
[obj performSelector:sel withObject:self]; //调用选择器方法  
}
```
使用`[[UIApplication sharedApplication] keyWindow]`查找应用程序的主窗口对象

----------

### Block 用法总结

> [Block 用法总结 (I)](http://www.cnblogs.com/lzz900201/archive/2013/04/17/3025340.html)

*Block是Apple Inc.为C、C++以及Objective-C添加的特性，使得这些语言可以用类lambda表达式的语法来创建闭包 ——维基百科*
*A block is an anonymous inline collection of code, and sometimes also called a "closure". ——Apple文档*
*闭包就是能够读取其它函数内部变量的函数。*

*在计算机程序设计中，回调函数，或简称回调（Callback），是指通过函数参数传递到其它代码的，某一块可执行代码的引用。这一设计允许了底层代码调用在高层定义的子程序。 ——维基百科*

回调就是说，我一个操作执行完成之后，提供给调用者一个接口，供调用者定义一些操作。obj-C的代理模式就是典型的回调。

```objc
@interface BlockTest : NSObject

-(void)testBlock1:(void(^)(void))finish;

@end

@implementation BlockTest

-(void)testBlock1:(void(^)(void))finish
{
    NSLog(@"test1");
    finish();
}

@end
```
这是一个很简单的block参数， 没有返回值， 没有参数,调用时
```objc
BlockTest *bt = [BlockTest new];
[bt testBlock1:^{
    NSLog(@"finish test");
}];
```
输出为
test1
finish test

带参数的回调函数
```objc
@interface BlockTest : NSObject

-(void)testBlock2:(void (^)(int a,int b))result;

@end

@implementation BlockTest

-(void)testBlock2:(void (^)(int a,int b))result
{
    NSLog(@"test2");
    int a = 1;
    int b = 2;
    result(a,b);
}

@end
```
调用时
```objc
[bt testBlock2:^(int a, int b) {
    NSLog(@"result: %d",a+b);
}];
```
类将自己内部的两个局部变量， 通过block传递给了调用者
结果为
test2
result: 3

block还可以有返回值，将block在调用者处执行的结果返回给方法本身
```objc
@interface BlockTest : NSObject

-(void)testBlock3:(int (^)(int a,int b))result;

@end

@implementation BlockTest

-(void)testBlock3:(int (^)(int a,int b))result
{
    NSLog(@"test3");
    int a = 1;
    int b = 2;
    int res = 0;
    if (result) {
        res = result(a,b);
    }
    NSLog(@"result: %d",res);
}

@end
```
调用此方法
```objc
[bt testBlock3:^int(int a, int b) {
    return  a+b;
}];
```
运行结果
test3
result: 3

