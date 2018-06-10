title: "Spring AOP相关知识点收集"
date: 2015-05-16 15:35:12
tags: ["Spring", "aop"]
categories: ["Java", "Spring"]
---

> [Spring 之AOP AspectJ切入点语法详解）](http://jinnianshilongnian.iteye.com/blog/1415606)
> [使用Spring进行面向切面编程](http://blog.csdn.net/yuqinying112/article/details/7336461)
> [aop:aspectj-autoproxy的内部机制](http://www.tuicool.com/articles/I3EFzm)
> [使用Spring的注解方式实现AOP](http://blog.csdn.net/a352193394/article/details/7345860)
> [Spring AOP 完成日志记录](http://hotstrong.iteye.com/blog/1330046)

### 语法
#### Spring AOP支持的AspectJ切入点指示符
切入点指示符用来指示切入点表达式目的，在Spring AOP中目前只有执行方法这一个连接点，Spring AOP支持的AspectJ切入点指示符如下：

* `execution`：用于匹配方法执行的连接点；
* `within`：用于匹配指定类型内的方法执行；
* `this`：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；
* `target`：用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；
* `args`：用于匹配当前执行的方法传入的参数为指定类型的执行方法；
* `@within`：用于匹配所以持有指定注解类型内的方法；
* `@target`：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；
* `@args`：用于匹配当前执行的方法传入的参数持有指定注解的执行；
* `@annotation`：用于匹配当前执行方法持有指定注解的方法；
* `bean`：Spring AOP扩展的，AspectJ没有对于指示符，用于匹配特定名称的Bean对象的执行方法；
* `reference pointcut`：表示引用其他命名切入点，只有@ApectJ风格支持，Schema风格不支持。

----------

#### 类型匹配语法
`*`：匹配任何数量字符；
`..`：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。
`+`：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。
```
java.lang.String    //匹配String类型；  
java.*.String   //匹配java包下的任何“一级子包”下的String类型；  
                //如匹配java.lang.String，但不匹配java.lang.ss.String  
java..*         //匹配java包及任何子包下的任何类型;  
                //如匹配java.lang.String、java.lang.annotation.Annotation  
java.lang.*ing  //匹配任何java.lang包下的以ing结尾的类型；  
java.lang.Number+ //匹配java.lang包下的任何Number的自类型；  
                  //如匹配java.lang.Integer，也匹配java.math.BigInteger 
```
匹配类型：使用如下方式匹配
```
注解？ 类的全限定名字
```

* 注解：可选，类型上持有的注解，如@Deprecated；
* 类的全限定名：必填，可以是任何类全限定名。

匹配方法执行：使用如下方式匹配
```
注解？ 修饰符? 返回值类型 类型声明?方法名(参数列表) 异常列表？
```

* 注解：可选，方法上持有的注解，如@Deprecated；
* 修饰符：可选，如public、protected；
* 返回值类型：必填，可以是任何类型模式；“*”表示所有类型；
* 类型声明：可选，可以是任何类型模式；
* 方法名：必填，可以使用“*”进行模式匹配；
* 参数列表：“()”表示方法没有任何参数；“(..)”表示匹配接受任意个参数的方法，“(..,java.lang.String)”表示匹配接受java.lang.String类型的参数结束，且其前边可以接受有任意个参数的方法；“(java.lang.String,..)” 表示匹配接受java.lang.String类型的参数开始，且其后边可以接受任意个参数的方法；“(*,java.lang.String)” 表示匹配接受java.lang.String类型的参数结束，且其前边接受有一个任意类型参数的方法；
* 异常列表：可选，以“throws 异常全限定名列表”声明，异常全限定名列表如有多个以“，”分割，如throws java.lang.IllegalArgumentException, java.lang.ArrayIndexOutOfBoundsException。

----------

#### 组合切入点表达式
AspectJ使用 且（&&）、或（||）、非（！）来组合切入点表达式。
在Schema风格下，由于在XML中使用“&&”需要使用转义字符`&amp;&amp;`来代替之，所以很不方便，因此Spring ASP 提供了and、or、not来代替&&、||、！。

----------

#### 切入点使用示例
##### execution
*使用“execution(方法表达式)”匹配方法执行-*

| 模式 | 描述 |
| --- | --- |
| `public * *(..)` | 任何公共方法的执行 |
| `* cn.javass..*.*(..)` | cn.javass包及所有子包下任何类的任何方法 |
| `* (!cn.javass..IPointcutService+).*(..)` | 非“cn.javass包及所有子包下IPointcutService接口及子类型”的任何方法 | 
| `@java.lang.Deprecated * *(..)` | 任何持有@java.lang.Deprecated注解的方法 |


##### within
*使用“within(类型表达式)”匹配指定类型内的方法执行；*

| 模式 | 描述 |
| --- | --- |
| within(cn.javass..*) | cn.javass包及子包下的任何方法执行 |
| within(cn.javass..IPointcutService+) | cn.javass包或所有子包下IPointcutService类型及子类型的任何方法 |
| within(@cn.javass..Secure *) | 持有cn.javass..Secure注解的任何类型的任何方法.必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 | 

##### this
*使用“this(类型全限定名)”匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口方法也可以匹配；注意this中使用的表达式必须是类型全限定名，不支持通配符*

| 模式 | 描述 |
| --- | --- |
| this(cn.spring.service.IPointcutService) | 当前AOP对象实现了 IPointcutService接口的任何方法 |
| this(cn.spring.service.IIntroductionService) | 当前AOP对象实现了 IIntroductionService接口的任何方法也可能是引入接口 |

##### target
*使用“target(类型全限定名)”匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；注意target中使用的表达式必须是类型全限定名，不支持通配符*

| 模式 | 描述 |
| --- | --- |
| target(cn.spring.service.IPointcutService) |  	当前目标对象（非AOP对象）实现了 IPointcutService接口的任何方法 |
| target(cn.spring.service.IIntroductionService) | 当前AOP对象实现了 IIntroductionService接口的任何方法也可能是引入接口 |

##### args
*使用“args(参数类型列表)”匹配当前执行的方法传入的参数为指定类型的执行方法；注意是匹配传入的参数类型，不是匹配方法签名的参数类型；参数类型列表中的参数必须是类型全限定名，通配符不支持；args属于动态切入点，这种切入点开销非常大，非特殊情况最好不要使用*

| 模式 | 描述 |
| --- | --- |
| args (java.io.Serializable,..) |  	任何一个以接受“传入参数类型为 java.io.Serializable” 开头，且其后可跟任意个任意类型的参数的方法执行，args指定的参数类型是在运行时动态匹配的 |

##### @within
*使用“@within(注解类型)”匹配所以持有指定注解类型内的方法；注解类型也必须是全限定类型名*

| 模式 | 描述 |
| --- | --- |
| @within cn.spring.Secure) | 任何目标对象对应的类型持有Secure注解的类方法；必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 |

##### @target
*使用“@target(注解类型)”匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；注解类型也必须是全限定类型名*

| 模式 | 描述 |
| --- | --- |
| @target(cn.spring.Secure) | 任何目标对象持有Secure注解的类方法；必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 |

##### @args
*使用“@args(注解列表)”匹配当前执行的方法传入的参数持有指定注解的执行；注解类型也必须是全限定类型名*

| 模式 | 描述 |
| --- | --- |
| @args (cn.spring.Secure) | 任何一个只接受一个参数的方法，且方法运行时传入的参数持有注解 cn.spring.Secure；动态切入点，类似于arg指示符； |

##### @annotation
*使用“@annotation(注解类型)”匹配当前执行方法持有指定注解的方法；注解类型也必须是全限定类型名*

| 模式 | 描述 |
| --- | --- |
| @annotation(cn.spring.Secure ) | 当前执行方法上持有注解 cn.spring.Secure将被匹配 |

##### bean
*使用“bean(Bean id或名字通配符)”匹配特定名称的Bean对象的执行方法；Spring ASP扩展的，在AspectJ中无相应概念*

| 模式 | 描述 |
| --- | --- |
| bean(*Service) | 匹配所有以Service命名（id或name）结尾的Bean |


----------

#### 通知参数

* 使用JoinPoint获取：Spring AOP提供使用org.aspectj.lang.JoinPoint类型获取连接点数据，任何通知方法的第一个参数都可以是JoinPoint(环绕通知是ProceedingJoinPoint，JoinPoint子类)，当然第一个参数位置也可以是JoinPoint.StaticPart类型，这个只返回连接点的静态部分。

1) `JoinPoint`：提供访问当前被通知方法的目标对象、代理对象、方法参数等数据
```java
package org.aspectj.lang;  
import org.aspectj.lang.reflect.SourceLocation;  
public interface JoinPoint {  
    String toString();         //连接点所在位置的相关信息  
    String toShortString();     //连接点所在位置的简短相关信息  
    String toLongString();     //连接点所在位置的全部相关信息  
    Object getThis();         //返回AOP代理对象  
    Object getTarget();       //返回目标对象  
    Object[] getArgs();       //返回被通知方法参数列表  
    Signature getSignature();  //返回当前连接点签名  
    SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置  
    String getKind();        //连接点类型  
    StaticPart getStaticPart(); //返回连接点静态部分  
}  
```
2）`ProceedingJoinPoint`：用于环绕通知，使用proceed()方法来执行目标方法
```java
public interface ProceedingJoinPoint extends JoinPoint {  
    public Object proceed() throws Throwable;  
    public Object proceed(Object[] args) throws Throwable;  
}
```
3) JoinPoint.StaticPart：提供访问连接点的静态部分，如被通知方法签名、连接点类型等
```java
public interface StaticPart {  
    Signature getSignature();    //返回当前连接点签名  
    String getKind();          //连接点类型  
    int getId();               //唯一标识  
    String toString();         //连接点所在位置的相关信息  
    String toShortString();     //连接点所在位置的简短相关信息  
    String toLongString();     //连接点所在位置的全部相关信息  
}
```

----------

### 实例

build.gradle
```gradle
apply plugin: 'java'
apply plugin: 'eclipse'

repositories {
	maven { 
	  url "http://repo.spring.io/libs-release"
	  url "http://maven.oschina.net/content/groups/public/"
	}
    mavenCentral()
}

dependencies {
	compile(
		[group: 'org.springframework', name: 'spring-aop', version: '4.1.5.RELEASE'],
		[group: 'org.springframework', name: 'spring-aspects', version: '4.1.5.RELEASE'],
		[group: 'org.springframework', name: 'spring-context-support', version: '4.1.5.RELEASE']
	)
}
```
service类
```java
public interface IHelloWorldService {
	public String sayHello(String s1,String s2);
}

public class HelloWorldService implements IHelloWorldService {
	@Override
	public String sayHello(String s1,String s2){
		return s1+" "+s2;
	}
}
```
aspect类
```java
@Aspect
public class MyAspect {
    //切入点 args获取方法参数
	@Pointcut(value="execution(* sayHello(..)) && args(s1,s2)")
	public void helloService(String s1,String s2){}
	//前置通知
	@Before(value="helloService(s1,s2)")
	public void beforeMethod(String s1,String s2){
		System.out.println("before:"+s1+" "+s2);
	}
	//环绕通知
	@Around("helloService(s1,s2)")
	public Object aroundMethod(ProceedingJoinPoint pjp,String s1,String s2) throws Throwable{
		System.out.println("around before: "+s1);
		Object obj = pjp.proceed();
		System.out.println("around after: "+s2);
		return obj;
	}
	//后置通知,returning为返回值
	@AfterReturning(value="helloService(s1,s2)",returning="rtv")
	public void afterMethod(JoinPoint jp,Object rtv,String s1,String s2){
	    //通过getArgs获取方法所有参数
		Object[] obj = jp.getArgs();
		for(Object o:obj){
			System.out.println(o.toString());
		}
		//获取类名及执行方法名
		System.out.println(jp.getSignature().getDeclaringTypeName()
		    +"."+jp.getSignature().getName()
		    +" return: "+rtv);
	}
	//@AfterThrowing当方法抛出异常后执行
	//@After类似于finally中的方法，不管方法有无异常抛出都会通知
}
```
configuration类
```java
@Configuration
@EnableAspectJAutoProxy     //开启aspect织入支持
public class AppConfig {
	@Bean
	public IHelloWorldService iHelloWorldService(){
		return new HelloWorldService();
	}
	@Bean
	public MyAspect myAspect(){
		return new MyAspect();
	}
}
```
test
```java
public class Test1 {
	private static AnnotationConfigApplicationContext ctx;
	public static void main(String[] args) {
		ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		IHelloWorldService service = ctx.getBean(IHelloWorldService.class);
		service.sayHello("a", "b");
	}
}
```
输出
```
around before: a
before:a b
around after: b
a
b
com.spring.service.IHelloWorldService.sayHello return: a b
```
