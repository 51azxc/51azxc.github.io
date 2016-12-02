title: "GSON相关知识"
date: 2015-03-31 15:59:08
tags: ["json", "gson"]
categories: "java"
---

> [JSON知识总结- Gson（四）List和Map ](http://blog.csdn.net/yuguiyang1990/article/details/8605407)
> [gson的@Expose注解和@SerializedName注解 ](http://blog.csdn.net/z69183787/article/details/18809815)
> [Json转换利器Gson之实例二-Gson注解和GsonBuilder](http://blog.csdn.net/lk_blog/article/details/7685190)
> [Gson使用二（GsonBuilder）](http://eksliang.iteye.com/blog/2175473)

### 隐藏属性
使用 `@Expose`注解来区分需要被实例化的属性。使用了此注解之后需要使用 `new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()`来创建对象，这样没有被标记`@Expose` 注释的属性将不会被序列化

```java
public class User {
  @Expose
  @SerializedName("user")   //重命名属性
  private String username;
  @Expose(serialize=true,deserialize=false)   //序列化时使用，反序列化不使用
  private String password;
  @since(1.1)    //根据版本来决定是否序列化/反序列化
  private String status;
  //get/set
  public static void main(String[] args) {
	User user = new User();
	user.setUsername("a");
	user.setPassword("b");
	user.setStatus("0");

	Gson g1 = new Gson();
	System.out.println(g1.toJson(user)); 	
	//result: {"user":"a","password":"b","status":"0"}		
	
	Gson g2 = new GsonBuilder()
	    .excludeFieldsWithoutExposeAnnotation() //没有标注@Expose注解的属性将不被导出
	    .create();
	System.out.println(g2.toJson(user));	
	//result:{"user":"a","password":"b"}
  }
}
```
使用`GsonBuilder()`方法构造实例，可以使用更多的配置，如：
```java
Gson g2 = new GsonBuilder()
    .enableComplexMapKeySerialization() //支持Map的key为复杂对象的形式
    .serializeNulls() //当需要序列化的值为空时，采用null映射，否则会把该字段省略
    .setDateFormat("yyyy-MM-dd HH:mm:ss") //转换日期格式
    .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)//段首字母大写,如使用@SerializedName注解的不会生效
    .setPrettyPrinting() //对json结果格式化
    .setVersion(1.2)    //对于标注@Since注解的字段来决定版本是否对该字段进行序列化/反序列化，对于标注@Until注解的字段来决定当前版本是否删除该字段
    .create();
```

----

### Gson对Map的转换

如果Map中包含了自定义类可以使用 `Type type = new TypeToken<Map<String , Class>>() {}.getType();` 来转换

```java
Type type = new TypeToken<Map<String , User>>() {}.getType(); 
Map<String, User> m = gson.fromJson(s1, type);
for(String key: m.keySet()){
  System.out.println(key+" : "+m.get(key).getUsername());
}
``` 
