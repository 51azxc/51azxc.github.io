title: "Java Servlet"
date: 2015-04-24 22:17:37
tags: ["servlet", "filter", "annotation"]
categories: "java"
---

> [Servlet3.0中Servlet的使用](http://haohaoxuexi.iteye.com/blog/2013691)
> [Servlet 3.0特性详解](http://blog.csdn.net/flfna/article/details/5598201)

#### 异步
Servlet 3.0之前，一个普通Servlet的主要工作流程大致如下：首先，Servlet接收到请求之后，可能需要对请求携带的数据进行一些预处理；接着，调用业务接口的某些方法，以完成业务处理；最后，根据处理的结果提交响应，Servlet线程结束。其中第二步的业务处理通常是最耗时的，这主要体现在数据库操作，以及其它的跨网络调用等，在此过程中，Servlet线程一直处于阻塞状态，直到业务方法执行完毕。在处理业务的过程中，Servlet资源一直被占用而得不到释放，对于并发较大的应用，这有可能造成性能的瓶颈。
现在通过使用Servlet 3.0的异步处理支持，之前的Servlet处理流程可以调整为如下的过程：**首先**，Servlet接收到请求之后，可能首先需要对请求携带的数据进行一些预处理；**接着**，Servlet线程将请求转交给一个异步线程来执行业务处理，线程本身返回至容器，此时Servlet还没有生成响应数据，异步线程处理完业务以后，可以直接生成响应数据（异步线程拥有`ServletRequest`和`ServletResponse`对象的引用），或者将请求继续转发给其它 Servlet。**如此一来**，Servlet线程不再是一直处于阻塞状态以等待业务逻辑的处理，而是启动异步线程之后可以立即返回。
对于一个Servlet如果要支持异步调用的话我们必须指定其`asyncSupported`属性为**true**（默认是**false**）。使用`@WebServlet`注解标注的Servlet我们可以直接指定其`asyncSupported`属性的值为**true**，如：
`@WebServlet(value=”/servlet/async”, asyncSupported=true)`。而对于在web.xml文件中进行配置的Servlet来说，我们需要在配置的时候指定其`asyncSupported`属性为**true**。
```xml
<servlet>  
   <servlet-name>xxx</servlet-name>  
   <servlet-class>xxx</servlet-class>  
   <async-supported>true</async-supported>  
</servlet>  
<servlet-mapping>  
   <servlet-name>xxx</servlet-name>  
   <url-pattern>xxx</url-pattern>  
</servlet-mapping>
```
Servlet的异步调用程序的关键是要调用当前`HttpServletRequest的startAsync()`方法。至于利用返回的AsyncContext来新起一个线程进行异步处理就不是那么的必须了，因为在HttpServletRequest startAsync()之后，我们可以自己新起线程进行异步处理。
除此之外，Servlet 3.0还为异步处理提供了一个监听器，使用**AsyncListener**接口表示。它可以监控如下四种事件：

1.异步线程开始时，调用**AsyncListener**的`onStartAsync(AsyncEventevent)`方法；
2.异步线程出错时，调用**AsyncListener**的`onError(AsyncEventevent)`方法；
3.异步线程执行超时，则调用**AsyncListener**的`onTimeout(AsyncEventevent)`方法；
4.异步执行完毕时，调用**AsyncListener**的`onComplete(AsyncEventevent)`方法；

要注册一个`AsyncListener`，只需将准备好的`AsyncListener`对象传递给`AsyncContext`对象的`addListener()`方法即可
```java
@WebServlet(urlPatterns={"/async"},asyncSupported=true)
public class AsyncServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		doPost(req, resp);
	}
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		resp.setContentType("text/plain;charset=UTF-8");
		PrintWriter pw = resp.getWriter();
		pw.write("begin");
		pw.flush();
		//开始异步调用
		AsyncContext context = req.startAsync();
		//设置当前异步调用对应的监听器
		context.addListener(new AsyncListener() {
			@Override
			public void onTimeout(AsyncEvent arg0) throws IOException {
				System.out.println("onTimeout");
			}
			@Override
			public void onStartAsync(AsyncEvent arg0) throws IOException {
				System.out.println("onStartAsync");
			}
			@Override
			public void onError(AsyncEvent arg0) throws IOException {
				System.out.println("onError");
			}
			@Override
			public void onComplete(AsyncEvent arg0) throws IOException {
				System.out.println("onComplete");
			}
		});
		//设置超时时间，当超时之后程序会尝试重新执行异步任务，即我们新起的线程
		context.setTimeout(1000000);
		//新起线程开始异步调用，start方法不是阻塞式的，它会新起一个线程来启动Runnable接口，之后主程序会继续执行
		context.start(new Runnable() {
			@Override
			public void run() {
				try{
				    //等待2秒
					Thread.sleep(2*1000);
					pw.write("async");
					pw.flush();
					//异步调用完成，如果异步调用完成后不调用complete()方法的话，异步调用的结果需要等到设置的超时时间过了之后才能返回到客户端
					context.complete();
				}catch(Exception e){
					e.printStackTrace();
				}
			}
		});
		pw.write("finish");
		pw.flush();
	}
}
```

----

##### 注解支持
###### `@WebServlet`注解
Servlet3.0支持使用注解配置Servlet。我们只需在Servlet对应的类上使用`@WebServlet`进行标注，我们就可以访问到该Servlet了，而不需要再在web.xml文件中进行配置。它有如下属性（以下所有属性均为可选属性，但是**vlaue**或者**urlPatterns**通常是必需的，且二者不能共存，如果同时指定，通常是忽略**value**的取值）:
| 属性名 | 类型 | 说明 |
| ------ | ---- | ---- |
| name | String | 指定Servlet的name属性，等价于`<servlet-name>`。如果没有显式指定，则该Servlet的取值即为类的全限定名 |
| value | String[] | 该属性等价于urlPatterns属性。两个属性不能同时使用 |
| urlPatterns | String[] | 指定一组Servlet的URL匹配模式。等价于`<url-pattern>`标签 |
| loadOnStartup | int | 指定Servlet的加载顺序，等价于`<load-on-startup>`标签 |
| initParams | WebInitParam[] | 指定一组Servlet初始化参数，等价于`<init-param>`标签 |
| asyncSupported | boolean | 声明Servlet是否支持异步操作模式，等价于`<async-supported>`标签 |
| description | String | 该Servlet的描述信息，等价于`<description>`标签 |
| displayName | String | 该Servlet的显示名，通常配合工具使用，等价于<display-name>标签 |

示例：
```java
@WebServlet(urlPatterns={"/simple"},
            asyncSupported=true,
            loadOnStartup=-1,
            name="SimpleServlet",
            displayName="ss",
            initParams={@WebInitParam(name="username",value="tom")} 
)  
```
如此一来，不必在web.xml中配置即可通过路径访问到相关servlet

----

###### `@WebInitParam`注解

**`@WebInitParam`**通常不单独使用，而是配合`@WebServlet`或者`@WebFilter`使用。它的作用是为Servlet或者过滤器指定初始化参数，这等价于web.xml中`<servlet>`和`<filter>`的`<init-param>`子标签
`@WebInitParam`具有下表给出的一些常用属性：

| 属性名 | 类型 | 可选 | 描述 |
| ------ | ---- | ---- | ---- |
| name | String | 否 | 指定参数的名字，等价于`<param-name>` |
| value | String | 否 | 指定参数的值，等价于`<param-value>` |
| description | String | 是 | 关于参数的描述，等价于`<description>` |


----

###### `@WebFilter`注解

`@WebFilter`用于将一个类声明为过滤器，该注解将会在部署时被容器处理，容器将根据具体的属性配置将相应的类部署为过滤器。该注解具有下表给出的一些常用属性(以下所有属性均为可选属性，但是**value**、**urlPatterns**、**servletNames**三者必需至少包含一个，且**value** 和**urlPatterns**不能共存，如果同时指定，通常忽略**value**的取值)

| 属性名 | 类型 | 说明 |
| ------ | ---- | ---- |
| filterName | String | 指定过滤器的name属性，等价于`<filter-name>` |
| value | String[] | 该属性等价于urlPatterns属性。两个属性不能同时使用 |
| urlPatterns | String[] | 指定一组过滤器的URL匹配模式。等价于`<url-pattern>`标签 |
| servletNames | String[] | 指定过滤器将应用于哪些Servlet。取值是`@WebServlet`中的**name**属性的取值，或者是**web.xml**中`<servlet-name>`的取值 |
| dispatcherTypes | DispatcherType | 指定过滤器的转发模式。具体取值包括`ASYNC`,`ERROR`,`FORWARD`,`INCLUDE`,`REQUEST` |
| initParams | WebInitParam[] | 指定一组Servlet初始化参数，等价于`<init-param>`标签 |
| asyncSupported | boolean | 声明过滤器是否支持异步操作模式，等价于`<async-supported>`标签 |
| description | String | 该过滤器的描述信息，等价于`<description>`标签 |
| displayName | String | 该过滤器的显示名，通常配合工具使用，等价于<display-name>标签 |

示例：
```java
@WebFilter(filterName="WebAppFilter",urlPatterns={"/*"})
```

----

###### `@WebListener`注解

`@WebListener`用于将类声明为监听器，被`@WebListener`标注的类必须实现以下至少一个接口
1. ServletContextListener 
2. ServletContextAttributeListener 
3. ServletRequestListener 
4. ServletRequestAttributeListener 
5. HttpSessionListener 
6. HttpSessionAttributeListener

该注解只有一个属性`value`用于描述监听器的信息
示例：
```java
@WebListener
public class BaseListener extends HttpServlet implements ServletContextListener{
  
}
```

----

###### `@MultipartConfig`注解

`@MultipartConfig`提供了对上传文件的支持。该注解标注在Servlet上面，以表示该Servlet希望处理的请求的 MIME类型是`multipart/form-data`。另外，它还提供了若干属性用于简化对上传文件的处理。具体如下：

| 属性名 | 类型 | 可选 | 描述 |
| ------ | ---- | ---- | ---- |
| fileSizeThreshold | int | 是 | 当数据量大于该值时，内容将被写入文件 |
| location | String | 是 | 存放生成的文件地址 |
| maxFileSize | long | 是 | 允许上传的文件最大值。默认值为-1，表示没有限制 |
| maxRequestSize | long | 是 | 针对该multipart/form-data请求的最大数量，默认值为-1，表示没有限制 |


----

#### Filter

> [Servlet3.0 Filter](http://takeme.iteye.com/blog/1961406)
> [	Servlet中的filter过滤器](http://my.oschina.net/flynewton/blog/12483)

```java
@WebFilter(filterName="WebAppFilter",urlPatterns={"/*"})
public class WebAppFilter implements Filter {
	
    public WebAppFilter() {}

	public void destroy() {}

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		HttpServletRequest req = (HttpServletRequest) request;
		HttpServletResponse res = (HttpServletResponse) response;
		// 不过滤的uri  
		String[] notFilter = new String[]{"log","resources"};
		// 请求的uri  
		String uri = req.getRequestURI();
		// 是否过滤  
		boolean doFilter = true;
		for(String s : notFilter){
			if(uri.indexOf(s) != -1){
				// 如果uri中包含不过滤的uri，则不进行过滤  
				doFilter = false;
				break;
			}
		}
		if(doFilter){
			// 执行过滤 
			
		}else{
			chain.doFilter(request, response);
		}
		
	}

	public void init(FilterConfig fConfig) throws ServletException { }

}
```

