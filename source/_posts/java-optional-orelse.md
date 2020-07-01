---
title: Optional.orElse与Optional.orElseGet的区别
date: 2020-06-22 17:33:41
tags: ["Java"]
categories: ["Java"]
---

**Java8**加入了`Optional`这一利器用来对付`null`。它里边就包含了`orElse`与`orElseGet`方法。
单看方法及注释的话，它俩的区别就是接收的参数不一样，以及一个返回`null`,另一个则不允许。
不过最近在使用过程中发现它俩还有一个区别，就是*无论`Optional`中的值为不为`null`，
`orElse`都会有执行，而`orElseGet`则不会执行。*

下面通过一个简单的代码段来测试一下：
```java
public static String other() {
    System.out.println("call other method");
    return "nothing";
}

public static String orElseMethod(String s) {
    return Optional.ofNullable(s).orElse(other());
}

public static String orElseGetMethod(String s) {
    return Optional.ofNullable(s).orElseGet(() -> other());
}

public static void main(String[] args) throws Exception {
    String s = null;
    System.out.println("----------orElse---------");
    System.out.println(orElseMethod(s));
    System.out.println("----------orElseGet---------");
    System.out.println(orElseGetMethod(s));
}

output:
----------orElse---------
call other method
nothing
----------orElseGet---------
call other method
nothing
```
查看输出信息可以得知，当`Optional`值为`null`的时候，`orElse`及`orElseGet`都执行了对应的方法并且返回了候补值。
当传入的值不为`null`时：
```java
public static void main(String[] args) throws Exception {
    String s = "a";
    System.out.println("----------orElse---------");
    System.out.println(orElseMethod(s));
    System.out.println("----------orElseGet---------");
    System.out.println(orElseGetMethod(s));
}

output:
----------orElse---------
call other method
a
----------orElseGet---------
a
```
这时`orElseGet`里的方法并不会执行，而`orElse`里的方法还是会执行。

乍看之下，它俩有点像单例模式中的*饿汉模式*与*懒汉模式*，一个是事先准备好返回值，
另一个则是需要用到的时候才去构造对应的返回值。因此这样看某些情况下`orElseGet`比`orElse`性能会好一些？

跑个分看看:
```java
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@State(Scope.Benchmark)
public class OptionalTest {

    @Param({"sss"})
    public String s = null;

    public String other() {
        return "nothing";
    }

    @Benchmark
    public String orElseMethod() {
        return Optional.ofNullable(s).orElse(other());
    }

    @Benchmark
    public String orElseGetMethod() {
        return Optional.ofNullable(s).orElseGet(() -> other());
    }

    public static void main(String[] args) throws Exception {
        Options options = new OptionsBuilder().include(OptionalTest.class.getSimpleName())
                .forks(2)
                .warmupIterations(4).warmupTime(TimeValue.seconds(1))
                .measurementIterations(10).measurementTime(TimeValue.seconds(1))
                .threads(4)
                .build();
        new Runner(options).run();
    }
}

output:
Benchmark                     (s)   Mode  Cnt    Score    Error   Units
OptionalTest.orElseGetMethod  sss  thrpt   40  494.108 ± 16.033  ops/us
OptionalTest.orElseMethod     sss  thrpt   40  461.837 ± 36.228  ops/us
```
这里的评测指标为**吞吐量**`Throughput`,表示指定时间内能执行多少次调用。那自然分数是越高越好。
从这个测试结果来看，`orElseGet`是比`orElse`分数高的，当然测试用例的不同分数自然不一样，此次跑分结果仅供娱乐。