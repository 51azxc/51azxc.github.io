title: "Java 项目获取路径"
date: 2015-04-11 14:30:45
tags: ["servlet", "jsp", "path"]
categories: "java"
---

> [JAVA获取当前工程路径(非web工程)](http://www.cnblogs.com/noures/archive/2012/08/16/2642349.html)
> [web项目中各种路径的获取](http://pengshao.iteye.com/blog/616342)
> [JAVA，JSP，Servlet获取当前工程路径-绝对路径](http://blog.csdn.net/sanyuesan0000/article/details/7433680)
> [ServletContext对象读取资源路径的三种方式](http://blog.csdn.net/a623397674a/article/details/14523177)
> [java类获取web应用的根目录（转载）](http://zch198627.iteye.com/blog/143399)
> [普通JAVA获取WEB项目下的WEB-INF目录](http://she.iteye.com/blog/1199723)

### Java普通项目获取路径

* 通过user.dir来获取
```java
System.out.println(System.getProperty("user.dir"));
```

* 利用File的函数获取
```java
File f = new File("");
try{
    //这里两者一样，如果上面File("")的路径为..，则此函数获取的为当前目录的父目录路径，下面的函数则还是当前目录
	System.out.println(f.getCanonicalPath());
	System.out.println(f.getAbsolutePath());
}catch(Exception e){
	e.printStackTrace();
}
```

* 获取编译后的classes文件所在的绝对路径
```java
System.out.println(PathTest.class.getResource(""));
```

### Servlet获取文件路径

```java
//获取当前项目名
System.out.println(request.getContextPath());
//获取到当前Servlet相对路径
System.out.println(request.getRequestURI());
//获取浏览器地址栏的路径
System.out.println(request.getRequestURL());
//获取当前Servlet的访问路径
System.out.println(request.getServletPath());
//获取服务器项目实际路径
System.out.println(request.getSession().getServletContext().getRealPath(""));
```

通过ServletContext获取资源文件
```java
//通过ServletContext获取资源文件
ServletContext context = this.getServletContext();
context.getResourceAsStream("/WEB-INF/classes/log4j.properties");
URL url = context.getResource("/WEB-INF/classes/log4j.properties");
System.out.println(url.getPath());
System.out.println(url.getFile());
```

### JSP获取文件路径
```java
<%=request.getContextPath() %>
<br />
<%=request.getServletPath() %>
<br />
<%=request.getRequestURI() %>
<br />
<%=request.getRequestURL().toString() %>
```

----

__System.getProperty()中的部分参数__
| 参数        | 说明   |
| --------   | :-----  |
| file.separator | 文件目录分割符 |
| java.home | jdk所在目录 |
| java.io.tmpdir | 临时文件目录 |
| java.library.path | 系统path参数 |
| java.version  | java 版本 |
| line.separator | 系统默认换行符 |
| path.separator | 路径分隔符 |
| os.name | 系统名 |
| os.arch | 系统指令集 |
| os.version | 系统内核版本 |
| user.name | 当前用户名 |
| user.home | 当前用户目录 |
| user.dir | 当前工作空间路径 |