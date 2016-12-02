title: "Objective-C 相关知识"
date: 2015-04-22 17:21:51
tags: "ivar"
categories: ["iOS","Objective-C"]
---

### Objective-C 基础

#### 声明变量

> [Object-C @property 小结](http://blog.csdn.net/dfqin/article/details/11669993)
> [IOS，objective_C中用@interface和 @property 方式声明变量的区别](http://www.cnblogs.com/letmefly/archive/2012/07/20/2601338.html)
> [iOS开发中常见的语句@synthesize obj=_obj的意义详解](http://moto0421.iteye.com/blog/1577459)
> [iOS中属性与成员变量的区别](http://www.cnblogs.com/ygm900/p/3660364.html)

##### `@property`是什么
标识为`@property`的变量可以自动生成`setter`与`getter`方法。
 __声明__: 声明格式为`@property (attributes) type propertyName`
__实现__: 在.m文件中的implements下，格式为`@synthsize name`即可。而在xcode4.4以后的版本，系统会自动合成, 等价于自己写了代码“ @synthesize  name = _name;” 。 也就是说如果没有特殊需求，只需要在头文件中声明而无需实现，就可以直接使用了
__成员变量访问权限__: 头文件中声明的成员变量，默认是`protected`，.m文件中声明的成员变量，默认是`private`的。合成属性时，`@synthesize  propertyName = _name`；如果变量`_name`没有声明，系统会自动生成该成员变量且为`private`权限。如`果_name`已声明，它们会自动合成

声明一个属性，如果没有声明为只读的，它默认会生成两个方法 `- (type)name` 和 `- (void)setName`; 为了可读性等其它原因，也可以改变属性的setter和getter访问名称
```objc
@property  (setter=setMyValue, getter=getBool) NSInteger  value;
```
这样的话就可以通过 `[obj setMyValue:10]` 和 `[obj getBool]`方法业访问成员变量了，此时`setValue`方法会被覆盖，不再存在

__@property的修饰属性(attributes)__

* 可读性：`readwrite` / `readonly` ，不写的话默认为readwrite，即会合成setter和getter方法
* 内存 `assign` / `retain` / `weak` / `strong` ，在非ARC环境下， assign为默认，引用计数不变；retain引用计数加1；在引用计数环境下，默认为strong，与retain作用相同；从5.0系统后引入了weak，作用与assign相似，不过当所指向对象引用为0时，自动置为nil

----

#### Selector基本概念

> [Selector基本概念和操作](http://moto0421.iteye.com/blog/1625204)

`@selector()`就是取类方法的编号,他的行为基本可以等同C语言的中函数指针,只不过C语言中，可以把函数名直接赋给一个函数指针，而Objective-C的类不能直接应用函数指针，这样只能做一个@selector语法来取。它的结果是一个SEL类型。这个类型本质是类方法的编号(函数地址)?

----

#### 引入头文件关键字`@Class`/`#import`

> [IOS开发技术之──头文件引用（@class/#import/#include）](http://blog.csdn.net/pjk1129/article/details/6590282)
> [IOS基础：深入理解Objective-c中@class的含义](http://www.cnblogs.com/martin1009/archive/2012/06/24/2560218.html)

`#include`:引入C头文件
`#import`:引入Objective-C的头文件
`@class`:类引用

`#import`确定一个文件只能被导入一次，这使你在递归包含中不会出现问题.`#import`比起`#include`的好处就是不会引起交叉编译.
`#import`方式会包含被引用类的所有信息，包括被引用类的变量和方法；`@class`方式只是告诉编译器在被引用类只是类的声明，具体这个类里有什么信息，这里不需要知道，等实现文件中真正要用到时，才会真正去查看被引用类中信息。
使用`@class`方式由于只需要只要被引用类的名称就可以了，而在实现类由于要用到被引用类中的实体变量和方法，所以需要使用`#import`来包含被引用类的头文件
`@class`是放在`interface`中的，只是在引用一个类，将这个被引用类作为一个类型，在实现文件中，如果需要引用到被引用类的实体变量或者方法时，还需要使用`#import`方式引入被引用类

----

#### ivar是什么意思

> [ObjectiveC基础－ivar是什么意思](http://blog.csdn.net/lvxiangan/article/details/18816481)

Objective-C运行时定义了几种重要的类型。

* **class**: 定义Objective-C类
* **ivar**: 定义对象的实例变量，包括类型与名字
* **protocol**: 定义协议
* **objc_property_t**: 定义属性
* **method**: 定义对象方法或者类方法。这个类型提供了方法的名字(*选择器*),参数类型与数量，返回值(合称为*方法签名*)，以及指向代码的函数指针(*方法的实现*)
* **SEL**: 定义选择器。选择器为方法名唯一的标识
* **IMP**: 定义方法的实现。这是一个指向某个函数的指针，该函数接受一个对象，一个选择器以及一个可变参数列表，返回一个对象。

----

#### Objective-C限定词

> [Objective-C 限定词 long short 等](http://blog.sina.com.cn/s/blog_7aa21f320100qugx.html)
> [objectiveC【语法】修饰符 static extern const](http://blog.csdn.net/xpwang168/article/details/8087143)

* **long**: 如果直接把限定词long放在int声明之前，那么所声明的整型变量在某些计算机上具有扩展的值域。例如`long int factorial`.这条语句将变量factorial声明为long的整型变量，也就是长整型。就象`float`和`double`变量一样，long变量的具体精度是由具体的计算机系统决定的。在许多系统上，`int`与`long int`具有相同的值域，而且任何一个都能存储4个字节(1个字节8位)，32位宽(2,147,483,647)的整型值。`long`在限定整型的时候，实际相当于双精度的`short`。
`long int`类型的常量值可通过在整型常量末尾添加字母L(大小写均可)来形成。单数字和L之间不允许由空格。因为小写的L和数字1容易混淆，建议有用到这种情况，都用大写。要用`NSLog`显示`long int`的值，使用字母l做为修饰符并放在整型格式符号i，o和x之前。例如“%lx”表示十六进制格式显示值。
当然，我们同样可以把`long`标识符放在`double`声明之前。`long double`常量可写成其尾部带有字母l或L的浮点常量。要显示`long double`的值，需要使用修饰符L。因此，`%Lf`用浮点计数法显示`long double`的值，`%Le`用科学计数法显示同样的值，而`%Lg`将告诉`NSLo`g在`%Lf`和`%Le`之间任选一个使用。

* **long long**: 双长整型相当于双精度long，可以用如下形式使用:
```
long long int maxAllowedStorage;
```
这条语句把指定的变量声明为具有特定扩展精度的双长整型变量，该扩展精度保证变量至少8个字节，具有64位的宽度。`NSLog`字符串不使用单个字母l，而使用两个l来显示`long long`的整数，例如“`%lli`”

* **short**: 把限定词`short`放在`int`声明之前时，它告诉Objective-C编译器要声明的特定变量用来存储相当小的整数。之所以使用`short`变量，主要原因是对节约内存空间的考虑，当程序员需要大量内存而可用的内存量又十分有限时，就可用`short`变量来解决这个问题。在某些计算机上，`short int`占用的内存空间是常规`int`变量所占空间的一半。在任何情况下，确保分配给`short int`的空间数量不少于2个字节，16位
**注意**，在Objective-C中，没有其他方法可显式地编写`short int`型常量。要显示`short int`变量，可将字母h放在任何普通的整型转换符之前，如`%hi`，`%ho`或`%hx`。换句话说，可用任何整型转换符号来显示`short int`，因为当它作为参数传递给`NSLog`例程时，可转换成整数

* **unsigned**: 这个最终限定词就是无符号，可放在int变量之前，当整数变量只用来存储正数的情况下使用最终限定符。以下语句
```
unsigned int counter;
```
向编译器声明：变量counter只用来保存正值。通过限制整型变量的使用，使它专门存储正整数，可以扩展整型变量的精度。一般`unsigned int`可简写为`uint`。

* **signed**: `signed`限定词可明确地告诉编译器特定变量是有符号的。它主要用在char声明前面。

* **const**: 修饰的东西不能被修改。指针类型根据位置的不同可以理解成3种情况:
1.常量指针
```objc
// 初始化之后不能赋值，指向的对象可以是任意对象，对象可变。
NSString * const pt1;
```
2.指向常量的指针
```objc
// 初始化之后可以赋值，即指向别的常量，指针本身的值可以修改，指向的值不能修改
const NSString * pt2;
```
3.指向常量的常量指针
```objc
const NSString *  const pt3;
```

* **extern**: 等同于c，全局变量的定义
```objc
//x .h 声明
extern const NSString * AA;

//x .m 定义
const NSString * AA = @"abc";

// 调用
#import "x.h"
或者再次申明
extern const NSString * AA;
```

* **static**: 等同于c，将变量的作用域限定于本文件,`static`变量属于本类，不同的类对应的是不同的对象;`static`变量同一个类所有对象中共享，只初始化一次。
1. `static const`变量同`static`的结论，只是不能修改了，但是还是不同的对象
2. `extern const`变量只有一个对象，标准的常量的定义方法
3. `extern`的意思就是这个变量已经定义了，你只负责用就行了
