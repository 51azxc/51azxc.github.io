title: "Java 集合相关知识"
date: 2015-04-20 23:36:49
tags: "collection"
categories: "java"
---

#### Map

##### 遍历MAP

> [java 遍历map 方法](http://www.360doc.com/content/10/0512/09/908129_27177692.shtml)
> [HashMap获取键值的用法](http://jackie9305.iteye.com/blog/341925)

```java
Map<String,String> map = new HashMap<String, String>();
map.put("1", "a");
map.put("2", "s");
map.put("3", "d");
//1. 利用entrySet遍历
Iterator it1 = map.entrySet().iterator();
while(it1.hasNext()){
	Map.Entry<String, String> mn = ((Entry<String, String>) it1.next());
	String key = mn.getKey();
	String value = mn.getValue();
	System.out.println("key: "+key+" value: "+value);
}
//2.1  先遍历key值
for(Iterator it2 = map.keySet().iterator(); it2.hasNext();){
	Object o = it2.next();
	System.out.println("key: "+o);
}
//2.2 遍历value值
for(Iterator it3 = map.values().iterator(); it3.hasNext();){
	Object o = it3.next();
	System.out.println("value: "+o);
}
//3.利用keySet遍历
for(Object o:map.keySet()){
	System.out.println("key: "+o+" value: "+map.get(o));
}
```

----

#### List

##### list<String>与String数组互转

> [Java 类型 ArrayList<String> 与类型 String[] 的双向转换](http://www.blogjava.net/shisanfeng/articles/191738.html)

```java
String[] ss = new String[]{"a","d","c","b"};
//String[] -> list<String>
ArrayList<String> list = new ArrayList<String>();
for(int i=0,len=ss.length; i<len; i++){
	list.add(ss[i]);
}
//or
ArrayList<String> list2 = Arrays.asList(list);
//list<String> -> String[]
String[] lists = new String[list.size()];
list.toArray(lists);
//给数组排序
Arrays.sort(lists);
```

----

list排序

> [JAVA对ArrayList排序](http://www.cnblogs.com/fzzl/archive/2010/08/14/1799408.html)
> [Java排序对象：Comparable和Compartor](http://www.360doc.com/content/11/1228/15/7489308_175599827.shtml)

* 使用Comparator接口实现排序
```java
class Student{
	String name;
	int age;
	public Student(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
Comparator<Student> comparator = new Comparator<Student>() {
	public int compare(Student o1, Student o2) {
		if(!o1.getName().equals(o2.getName())){
			//大于0则为o1属性大于o2属性，小于0则相反，等于0则相等
			return o1.getName().compareTo(o2.getName());
		}else if(o1.getAge()!=o2.getAge()){
			//名字不相等则按年龄排序
			return o1.getAge() - o2.getAge();
		}
		return 0;
	}
};

Student s1 = new Student("a",13);
Student s2 = new Student("b",11);
ArrayList<Student> sl = new ArrayList<Student>();
sl.add(s1);
sl.add(s2);
//排序
Collections.sort(sl,comparator);
```

* 需排序类实现Comparable接口
```java
class Student implements Comparable<Student>{
    String name;
	int age;
	public Student(String name, int age) {
		this.name = name;
		this.age = age;
	}
	@Override
    public int compareTo(Student o) {
		if(!this.getName().equals(o.getName())){
			return this.getName().compareTo(o.getName());
		}else if(this.getAge()!=o.getAge()){
			return this.getAge() - o.getAge();
		}
		return 0;
	}
}

Student s1 = new Student("a",13);
Student s2 = new Student("b",11);
ArrayList<Student> sl = new ArrayList<Student>();
sl.add(s1);
sl.add(s2);
//通过默认方法进行排序
Collections.sort(sl);
```

----

##### list remove问题

> [java list remove的问题](http://yh1022.iteye.com/blog/316913)

每当list执行了`remove()`方法之后，`size()`都会缩小，如果此时进行正常遍历可能会抛出越界异常，这是应该倒序遍历。
