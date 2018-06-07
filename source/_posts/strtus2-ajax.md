title: "struts2零散知识点整理"
date: 2015-04-21 00:15:45
tags: "ajax"
categories: ["java", "struts2"]
---

平时使用Struts2碰到的一些问题的解决方法。

<!-- more -->

### Struts2使用Ajax

> [struts2使用ajax技术返回字符串](http://hi.baidu.com/suinmi/item/5de98cfb8da44eee1b111fa8)

1. 使用stream来进行服务端处理
action
```java
public class TextResult extends ActionSupport  {
    private InputStream inputStream;
    public InputStream getInputStream() {
        return inputStream;
    }

    public String execute() throws Exception {
        inputStream = new StringBufferInputStream("ok");
        return SUCCESS;
    }
} 
```
struts.xml
```xml
<action name="text-result" class="actions.TextResult">
    <result type="stream">
        <param name="contentType">text/html </param>
        <param name="inputName">inputStream </param>
    </result>
</action> 
```

2. 将struts2的标签theme属性设置为ajax
这种方法要求是服务端不需要特殊处理，就像一个单纯的action处理一样，你只需要在action中进行处理，
并转向到一个jsp文件中，
在发送ajax的页面中使用 <s:a>标签，将theme属性设置为ajax,并设置接收服务器返回信息的控件id.
```html
<s:a href="process.action" theme="ajax" targets="result" cssStyle="text-align:center;"/>
<div id="result"> </div> 
```

----

### `java.util.MissingResourceException Can't find bundle for base name`

> [解决方法：java.util.MissingResourceException Can't find bundle for base name](http://blog.chinaunix.net/uid-25820084-id-3494142.html)

这个问题的原因是`MessageResource_zh_CN.properties`，这个配置文件没有放在classpath中，
```
ResourceBundle config = ResourceBundle.getBundle("com.amaker.test.MessageResource");
```
要按照路径，把你的配置文件加入ClassPath中就可以了

----------

### 解决`java.lang.IllegalStateException`

> [Web开发中常见的java.lang.IllegalStateException错误](http://blog.sina.com.cn/s/blog_6151984a0100owod.html)

JSP文件或struts action(纯servlet应用中没发现此问题)中采用了,如下代码:
```java
public void print2Screen(HttpServletResponse resp,String encodeString,String[] htmlCommands) throws IOException{
    resp.setCharacterEncoding(encodeString);
    ServletOutputStream httpOutput= resp.getOutputStream();
    for(String temp:htmlCommands)
        httpOutput.write(temp.getBytes());
}
```
**深层原理**:
1. Servlet规范说明，不能既调用 `response.getOutputStream()`，又调用`response.getWriter()`，无论先调用哪一个，在调用第二个时候应会抛出 `IllegalStateException`.
2. servlet代码中有`out.write("")`，这个和JSP中缺省调用的`response.getOutputStream()`产生冲突.因为在jsp中，`out`变量是通过`response.getWriter`得到的，在程序中既用了 `response.getOutputStream`，又用了`out`变量，故出现以上错误。

**解决方法**

* 在JSP文件中,加入下面两句
```jsp
<%
out.clear();
out = pageContext.pushBody();
%>
```
此法的缺陷:
很多开发项目并不是JSP前端,如freemarker,velocity等
造成问题的`response.getOutputStream()`并未被写在JSP里,而是写在servlet/action里

* 在action中,不要return 回具体的result文件,而是`return null`

----

### Struts2国际化

> [国际化之struts2实现研究](http://blog.csdn.net/zollty/article/details/8710718)
> [Struts2 的国际化实现](http://www.cnblogs.com/lihuiyy/archive/2013/03/14/2958782.html)

在src目录下添加两个资源文件,格式`baseName_language_country.properties`例如
`message_zh_CN.properties`
```
login.title=请登录
login.username=用户名
login.password=密码
login.welcome=欢迎，{0}
```
`message_en_US.properties`
```
login.title=Please login
login.username=Username
login.password=Password
login.welcome=Welcome，{0}
```
在jsp中使用
```html
<s:text name="login.title"></s:text>
<s:textfield name="username" key="login.username"></s:textfield>
<s:textfield name="password" key="login.password"></s:textfield>
```
在action中使用
```java
getText("login.username");
//使用占位符
getText("login.welcome", "user");
```

在jsp中实行中英文切换
```html
<a href="login.action?request_locale=zh_CN">中文</a>
<a href="login.action?request_locale=en_US">English</a>
```

----

### Struts2中不同namespace的重定向用法

> [struts2中不同namespace的重定向用法](http://hi.baidu.com/wanglshen1/item/ab0c599a12b1a236326eeb50)

简单的同一个包或者namespace中：
```xml
<result name="success" type="redirectAction">xxx?a=1</result>
```

xxx 就是指你要重定向的action，?a=1 这个是参数的传递，你若是有参数的话可以这样传递
不同包或者不同namespace的
```xml
<result name="success" type="redirectAction">
  <param name="actionName">xxx</param>
  <param name="namespace">/xx</param>
</result>
```

xxx ：重定向的action
/xx: xxx所在的namespace

----

### struts2中 "#" "%" "$" 的区别和用法

> [struts2中 # % $ 的区别和用法](http://hi.baidu.com/golotus/item/6e16444df986d8e81f19bc1e)

表达式语言主要有以下几大好处：  
1. 避免`request.getAttribute()`和`myBean.getMyProperty()`之类的语句，使页面更简洁；  
2. 支持运算符（如+-*/），比普通的标志具有更高的自由度和更强的功能；  
3. 简单明了地表达代码逻辑，使用代码更可读与便于维护。

Struts2 中OGNL表达式的用法：
OGNL（Object-Graph Navigation Language），可以方便地操作对象属性的开源表达式语言；

“#”主要有三种用途：  

1. 用于过滤和投影（projecting)集合，如`books.{?#this.price<100}；`  
2. 构造Map，如`#{'foo1':'bar1', 'foo2':'bar2'}`。  
3. 访问OGNL上下文和Action上下文，#相当于`ActionContext.getContext()；`下表有几个ActionContext中有用的属性：  

| 名称 | 作用 | 例子 |
| ---- | ---- | ---- |
| parameters | 包含当前HTTP请求参数的Map | #parameters.id[0]作用相当于request.getParameter("id") |
| request | 包含当前HttpServletRequest的属性（attribute)的Map | #request.userName相当于request.getAttribute("userName") |
| session | 包含当前HttpSession的属性（attribute）的Map | #session.userName相当于session.getAttribute("userName") |
| application | 包含当前应用的ServletContext的属性（attribute）的Map | #application.userName相当于application.getAttribute("userName") |
| attr | 用于按request > session > application顺序访问其属性（attribute） | #attr.userName相当于按顺序在以上三个范围（scope）内读取userName属性，直到找到为止 |


“%”的用途是在标志的属性为字符串类型时，计算OGNL表达式的值。例如在Ognl.jsp中加入以下代码： 
```html
<h3>%的用途</h3>  
<p><s:url value="#foobar['foo1']" /></p>  
<p><s:url value="%{#foobar['foo1']}" /></p>  
```

“$”有两个主要的用途 ：    
   1. 用于在国际化资源文件中，引用OGNL表达式 
   2. 在Struts 2配置文件中，引用OGNL表达式，如
```xml
<action name="AddPhoto" class="addPhoto">  
  <interceptor-ref name="fileUploadStack" />              
  <result type="redirect">ListPhotos.action?albumId=${albumId}</result>  
</action>
```