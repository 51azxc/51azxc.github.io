title: "Jetty相关"
date: 2015-04-23 14:25:51
tags: ["jetty", "mvc"]
categories: ["java","jetty"]
---

> [Jetty/Tutorial/Embedding Jetty](https://wiki.eclipse.org/Jetty/Tutorial/Embedding_Jetty)
> [Java / Jetty: How to Add Filter to Embedded Jetty](http://stackoverflow.com/questions/19530806/java-jetty-how-to-add-filter-to-embedded-jetty)
> [EnumSet的几个例子](http://mouselearnjava.iteye.com/blog/2156221)

#### 使用代码运行jetty
运行一个war包
```java
public class OneWebApp
{
    public static void main(String[] args) throws Exception
    {
        String jetty_home = System.getProperty("jetty.home","..");
 
        Server server = new Server(8080);
 
        WebAppContext webapp = new WebAppContext();
        webapp.setContextPath("/");
        webapp.setWar(jetty_home+"/webapps/test.war");
        server.setHandler(webapp);
 
        server.start();
        server.join();
    }
}
```
运行一个web项目
```java
public class OneWebAppUnassembled
{
    public static void main(String[] args) throws Exception
    {
        Server server = new Server(8080);
 
        WebAppContext context = new WebAppContext();
        context.setDescriptor(webapp+"/WEB-INF/web.xml");
        context.setResourceBase("../test-jetty-webapp/src/main/webapp");
        context.setContextPath("/");
        context.setParentLoaderPriority(true);
 
        server.setHandler(context);
 
        server.start();
        server.join();
    }
}
```
运行spring mvc项目
```java
public static void main(String[] args) throws Exception {
		Server server = new Server(8090);
		WebAppContext context = new WebAppContext();
		//添加log4j监听器
		context.setInitParameter("log4jConfigLocation", "classpath:log4j.properties");
		context.addEventListener(new Log4jConfigListener());
		//设置访问路径
		context.setContextPath("/test1");
		//设置页面资源文件夹
		context.setResourceBase("../test1/src/main/webapp");
		EnumSet<DispatcherType> es = EnumSet.of(DispatcherType.ASYNC,DispatcherType.ERROR,DispatcherType.REQUEST,DispatcherType.FORWARD);
		//添加Filter
		context.addFilter(new FilterHolder(WebAppFilter.class), "/*", es);
		AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
		//加载spring注解配置类
		ctx.register(WebMvcConfig.class);
		DispatcherServlet ds = new DispatcherServlet(ctx);
		context.addServlet(new ServletHolder(ds), "/");
		//添加servlet
		context.addServlet(HelloServlet.class, "/helloServlet");
		
		server.setHandler(context);
		
		server.start();
		server.join();
}
```

----

#### jetty相关配置

> 来自
> [jetty配置文件详解](http://blog.csdn.net/fjslovejhl/article/details/15501091)
> [Jetty 的配置](http://www.cnblogs.com/shitou/archive/2011/05/30/2063423.html)

Jetty 的配置文件放在 etc 路径下,jetty.xml文件是默认的配置文件,jetty-jmx.xml是启动 JMX 控制的配置文件; jetty-plus.xm1文件是在增加 Jetty 扩展功能的配置文件。启动Jetty的命令为(配置环境路径或者进入Jetty目录下执行下面命令):
```bash
java -jar startup.jar
```
默认使用jetty.xm1文件时启动Jetty，即与如下命令效果相同
```bash
java -jar startup.jar etc/jetty.xml
```
启动时也可以指定多个配置文件，可输入如下命令
```bash
java -jar startup.jaretc/jetty.xml etc/jetty-plus.xml
```
打开 Jetty 配置文件，该配置文件的根元素是`Configure`，另外还会看到有如下的配
置元素。

* **Set**: 相当于调用 setxx 方法。
* **Get**: 相当于调用 getXxx 方法。
* **New**: 创建某个类的实例。
* **Arg**: 为方法或构造器传入参数。
* **Array**: 设置一个数组。
* **Item**: 设置数组或集合的-J页。
* **Call**: 调用某个方法。

Jetty 是个嵌入式 Web 容器，因此它的服务对应一个 `Server` 实例，可以看到配置文件中有如下片段:
```xml
<!--配置了一个Jetty服务器进程-->
<Configure id="Server" class="org.mortbay.jetty.Server">
```
上述是整个配置文件的root元素，读到它的时候会创建一个server对象，当然这个server对象的创建采用的是默认构造函数，因而可以理解为它是一个空的server


把相应的war包丢到jetty目录下的webapps目录里即可运行，在jetty目录下新建一个文件夹work，这样war包解压的文件就不会放到默认临时文件夹而放到work目录下

