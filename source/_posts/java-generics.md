title: "java 泛型"
date: 2015-04-25 16:03:39
tags:
categories: "java"
---

> [java 泛型详解](http://lichaozhangobj.iteye.com/blog/476911)

普通泛型
```java
class G1<T,V>{
	private T t;
	private V v;
	public T getT() {
		return t;
	}
	public void setT(T t) {
		this.t = t;
	}
	public V getV() {
		return v;
	}
	public void setV(V v) {
		this.v = v;
	}
}

public static void main(String[] args) {
  //设置T的类型为Integer,V的类型为String
  G1<Integer,String> g1 = new G1<Integer,String>();
  g1.setT(111);
  g1.setV("aaa");
  print1(g1);
}
//通过通配符可以获取任意类型的泛型
public static void print1(G1<?,?> g){
	System.out.println(g.getT()+" "+g.getV());
}
```

受限泛型
```java
//这里只接受G1<T,V>中T为Number及其子类的类型，V为String或Object类型
public static void print2(G1<? extends Number,? super String> g){
	System.out.println(g.getT()+" "+g.getV());
}
```

泛型数组
```java
//接受不限个数的参数
public static <T> T[] func1(T...args){
	return args;
}

public static <T> void func2(T ts[]){
	for(T t:ts){
		System.out.println(t);
	}
}

public static void main(String[] args) {
    String[] ss = func1("a","b","c","d");
	func2(ss);
}
```

泛型嵌套
```java
class G2<K>{
	private K k;
	public K getK() {
		return k;
	}
	public void setK(K k) {
		this.k = k;
	}
}

public static void main(String[] args) {
    G2<G1<Integer,String>> g2 = new G2<G1<Integer,String>>();
	g2.setK(g1);
	System.out.println(g2.getK().getV());
}

```