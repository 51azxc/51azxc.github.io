---
title: stream中去除相同的元素
date: 2020-06-24 19:39:51
tags: ["Java"]
categories: ["Java"]
---

给一个集合中的元素*去重*算是一个比较常见的任务需求了。这里记录一下通过`stream`来对元素去重操作的两种方法。

#### distinct方法
`stream`本身提供了`distinct()`这样的方法来对集合中的元素进行去重操作。不过默认情况下对于集合中的自定义类是无能为力的。
看了下关于`distinct()`中的注释，他是利用`equals()`方法来判断集合中的对象是否完全相等来去重的。
因此如果是自定义类，可以通过重写`equals()`方法来达到目的:
```java
class Student {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "{ name: \"" + name + "\", age: " + age + "\"}";
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) { return true; }
        if (obj == null || getClass() != obj.getClass()) { return false; }
        Student s = (Student)obj;
        if (this.name == null) {
            if (s.name != null) { return false; }
        } else if (!this.name.equals(s.name)) { return false; }
        if (this.age != s.age) { return false; }
        return true;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```
这里定义了一个Student类，然后通过判断name与age属性来判断对象是否相等。
需要注意的是，点开`equals()`方法会看到以下注释：
> 请注意，通常每当重写此方法时，都必须重写{@code hashCode}方法，以便维护{@code hashCode}方法的常规协定，该协定规定相等的对象必须具有相等的哈希码。

因此也不要忘记了重写`hashCode()`方法。接下来测试一下。
```java
List<Student> students = new Random().ints(10, 10, 16).boxed()
                .map(i -> new Student(String.valueOf((char)(55+i)), i)).collect(Collectors.toList());
students.forEach(System.out::println);
System.out.println("-----------------------------------");
students.stream().distinct().forEach(System.out::println);
```
输出的结果为:
```
{ name: "C", age: 12"}
{ name: "E", age: 14"}
{ name: "A", age: 10"}
{ name: "A", age: 10"}
{ name: "C", age: 12"}
{ name: "A", age: 10"}
{ name: "C", age: 12"}
{ name: "A", age: 10"}
{ name: "F", age: 15"}
{ name: "B", age: 11"}
-----------------------------------
{ name: "C", age: 12"}
{ name: "E", age: 14"}
{ name: "A", age: 10"}
{ name: "F", age: 15"}
{ name: "B", age: 11"}
```

可以看到`distinct()`方法已经生效了。


#### filter方法

如果不想修改自定义类的`equals()`与`hashCode()`方法，就可以利用`filter`方法来过滤掉重复的类。
`filter`方法接收的是一个`Predicate`,他可以筛选出在`stream`中与该`Predicate`匹配的元素，
因此可以写个去重的`Predicate`来达到目的：
```java
public static <T> Predicate<T> distinctByKeys(Function<? super T, ?>... keyExtractors) {
    Map<Object, Boolean> map = new ConcurrentHashMap<>();
    return t -> {
        List<?> keys = Arrays.stream(keyExtractors).map(k -> k.apply(t)).collect(Collectors.toList());
        return map.putIfAbsent(keys, Boolean.TRUE) == null;
    };
}
```
`Function`有点类似匿名函数，他通过传入一个参数，然后可以返回一个结果。
这里通过将这些`Function`的执行结果组成一个列表当作`Map`的key存入到指定的`Map`中，
`putIfAbset`方法与`put`方法不同点在于它并不会覆盖掉`Map`中已存在的键值对，并且还会返回该键所对应的值。
所以如果`Map`中没有该键值对时，必然返回的值为`null`。
因此第一次放入`Map`中的元素都能成功放入，返回的都是`null`,如果`Map`本身已包含了该元素，则不会返回`null`，说明该元素已经重复了。
这里主要是通过键来过滤元素，对值的要求不高，使用`Boolean.True`对象是因为它占用的字节数较少。
因为`stream`有可能通过`parallel()`方法来变成一个`parallelStream`,因此需要使用`ConcurrentHashMap`来确保多线程下的访问。
最终配合`filter()`方法可以过滤掉重复的元素。
```java
students.stream().filter(distinctByKeys(Student::getName, Student::getAge)).forEach(System.out::println);
```
最终输出结果与上边过滤结果一致。

如果只是想要利用类的单个属性来过滤，则直接将`Function`的执行结果作为key即可：
```java
public static <T> Predicate<T> distinctByKey(Function<? super T, Object> keyExtractor) {
    Map<Object, Boolean> map = new ConcurrentHashMap<>();
    return t -> map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
}

students.stream().filter(distinctByKey(Student::getName)).forEach(System.out::println);
```

