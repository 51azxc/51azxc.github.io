title: "Spring MVC部分知识点整理"
date: 2016-07-25 18:09:13
tags: ["Spring", "mvc", "gson", "Exception", "test"]
categories: ["Java", "Spring"]
---

### SpringMVC基于Servlet3无XML配置

> [一种maven改造快速支持servlet3.1web工程的方法](http://shmilyaw-hotmail-com.iteye.com/blog/2221134)

在`Servlet3`中，可以实现无`Web.xml`配置，需要一个类来实现`WebApplicationInitializer`接口

```java
public class WebInitializer implements WebApplicationInitializer {

	private static final String DISPATCHER_SERVLET_NAME = "dispatcher";
	
	public void onStartup(ServletContext servletContext) throws ServletException {
		AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
		//配置类，装载了SpringMVC的配置
		ctx.register(WebMvcConfig.class);
		ctx.setServletContext(servletContext);
		
		registerHiddenHttpMethodFilter(servletContext);
		
		Dynamic servlet = servletContext.addServlet(DISPATCHER_SERVLET_NAME, new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
	}
	
	private void registerHiddenHttpMethodFilter(ServletContext servletContext) {
        FilterRegistration.Dynamic fr = servletContext
                .addFilter("hiddenHttpMethodFilter", HiddenHttpMethodFilter.class);
        fr.addMappingForServletNames(
                EnumSet.of(DispatcherType.REQUEST, DispatcherType.FORWARD),
                false,
                DISPATCHER_SERVLET_NAME);
    }

}
```

如果要让`Maven`项目支持`Servlet3`的配置，需要修改的地方就有点多了
* 首先要让项目本身支持`Servlet3`，如果配置中的`Project Facets`不能做转换，可以到项目文件夹下找到一个隐藏文件夹`.settings`，然后找到`org.eclipse.wst.common.project.facet.core.xml`文件，打开并修改部分属性
```xml
<?xml version="1.0" encoding="UTF-8"?>
<faceted-project>
  <fixed facet="wst.jsdt.web"/>
  <!-- 改成合适的jdk版本 -->
  <installed facet="java" version="1.7"/>
  <installed facet="wst.jsdt.web" version="1.0"/>
  <!-- 改成版本为3.1，即支持Servlet3.1的配置 -->
  <installed facet="jst.web" version="3.1"/>
</faceted-project>
```
* 然后在项目文件`pom.xml`中的`<build>`节点下,`<finalName>`节点后添加如下配置
```xml
<plugins>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-compiler-plugin</artifactId>
		<version>3.1</version>
		<configuration>
			<source>1.7</source>
			<target>1.7</target>
		</configuration>
	</plugin>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-war-plugin</artifactId>
		<version>2.4</version>
		<configuration>
			<warSourceDirectory>src/main/webapp</warSourceDirectory>
			<failOnMissingWebXml>false</failOnMissingWebXml>
		</configuration>
	</plugin>
</plugins>
```
这样构建出来的项目就可以支持无XML风格的`Servlet3`项目了

----

### Spring MVC 静态资源报错

> [Spring MVC: Resources](http://fruzenshtein.com/spring-mvc-resources/)
> [Spring MVC with Java based config - 404 not found for static resources](http://stackoverflow.com/questions/22013074/spring-mvc-with-java-based-config-404-not-found-for-static-resources)
> [spring mvc 静态资源 404问题](http://blog.csdn.net/this_super/article/details/7884383)

在配置类中配置静态资源的访问路径:
```java
@Configuration
@EnableWebMvc
@ComponentScan
public class WebMvcConfig extends WebMvcConfigurerAdapter{
	@Bean 
	public ViewResolver viewResolver(){
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/page/");
		resolver.setSuffix(".jsp");
		return resolver;
	}
	
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //指定resources文件夹下任意目录,子目录的资源文件配置访问路径到resources
		registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
	}
}
```

如果页面是在项目的主目录下则可以使用如下路径访问:
```html
<link href="resources/css/main.css" rel="stylesheet" type="text/css" /> 
```
如果是在子集目录下,例如`http://localhost:8080/webtest1/index.html`，则需要添加一个层次路径:
```html
<link href="../resources/css/main.css" rel="stylesheet" type="text/css" /> 
```

也可使用`<c:url>`来引入静态文件:
```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<script src="<c:url value='/resources/js/jquery.js' />"></script>
```

----

### Spring 引入resources文件夹内的资源文件

> [Java (maven web app), getting full file path for file in resources folder?](http://stackoverflow.com/questions/7184186/java-maven-web-app-getting-full-file-path-for-file-in-resources-folder)

对于`Maven`或者`Gradle`项目架构，资源文件一般放置于`src/main/resources`文件夹中，如果需要读取到这个文件夹下的资源文件，可以借助`org.springframework.core.io.Resource`:
```java
//获取到resources下的config.properties文件
Resource resource = new ClassPathResource("config.properties");
File file = resource.getFile();
```

----

### Spring 访问路径@PathVariable包含"."号

> [Spring MVC @PathVariable with dot (.) is getting truncated](http://stackoverflow.com/questions/16332092/spring-mvc-pathvariable-with-dot-is-getting-truncated)

默认情况下，对于`@PathVariable`注解获得的路径变量会把特殊符号给去掉，如果想要包含特殊符号(如"."号)，需要使用正则表达式:
```java
@RequestMapping(value="/files/{filename:.+}", produces="application/json; charset=UTF-8")
public void checkFileInfo(@PathVariable String filename) {
    //例如filename = test1.xls
}
```

----

### 使用Gson作为Spring解析封装json的工具包

> [Configure Gson in Spring before using GsonHttpMessageConverter](http://stackoverflow.com/questions/31335146/configure-gson-in-spring-before-using-gsonhttpmessageconverter)

想要替换`Gson`包为**Spring**解析`json`的工具包，需要在配置类中重写对应方法：
```java
@Configuration
@EnableWebMvc
@ComponentScan
public class WebMvcConfig extends WebMvcConfigurerAdapter{
	@Override
	public void configureMessageConverters(
			List<HttpMessageConverter<?>> converters) {
		super.configureMessageConverters(converters);
		GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
		Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE).create();
		converter.setGson(gson);
		converters.add(converter);
	}
}
```
接下来在使用了`@ResponseBody`注解的方法，或者直接标注了`@RestController`控制类中有返回数据的方法将全部通过`Gson`来封装成`json`格式的数据。

----

### Spring MVC捕获异常

> [Spring MVC中的异常处理](http://ningandjiao.iteye.com/blog/1995270)

首先创建一个异常类:
```java
public class ResourceNotFoundException extends Exception {
	public ResourceNotFoundException(String message) {
		super(message);
	}
}
```
然后建立一个异常处理类:
```java
@ControllerAdvice
public class WopiExceptionHandler {
	@ResponseStatus(HttpStatus.NOT_FOUND)
	@ExceptionHandler(ResourceNotFoundException.class)
	@ResponseBody
	public String handlerResourceNotFound(ResourceNotFoundException ex) {
		return ex.getMessage();
	}
}
```
接下来在对应的控制类方法中抛出异常:
```java
@RequestMapping(value="/files/{fileId}", method=RequestMethod.GET, produces="application/json; charset=UTF-8")
@ResponseBody
public WopiFileInfo checkFileInfo(@PathVariable String fileId) throws Exception {
	String filePath = wopiService.getFilePath(fileId);
	if (CommonUtil.isEmptyString(filePath)) {
		throw new ResourceNotFoundException("Resource not found/user unauthorized");
	}
	...
}
```
测试一下
```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { TestConfig.class })
public class TestController {
	
	@Test
	public void testCheckFileInfo() throws Exception {
		mockMvc.perform(get("/files/{fileId}", "2"))
			.andExpect(status().isNotFound())
			.andExpect(content().string("\"Resource not found/user unauthorized\""));
	}
}
```