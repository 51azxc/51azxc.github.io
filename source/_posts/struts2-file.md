title: "Struts2 部分问题"
date: 2015-05-09 22:19:52
tags: ["i18n","upload","download"]
categories: ["java", "struts2"]
---

### `java.util.MissingResourceException Can't find bundle for base name`

> [解决方法：java.util.MissingResourceException Can't find bundle for base name](http://blog.chinaunix.net/uid-25820084-id-3494142.html)

这个问题的原因是`MessageResource_zh_CN.properties`，这个配置文件没有放在classpath中，
```
ResourceBundle config = ResourceBundle.getBundle("com.amaker.test.MessageResource");
```
要按照路径，把你的配置文件加入ClassPath中就可以了


----------

### `java.lang.IllegalStateException`

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

----------

### struts2文件上传下载

> [struts2文件上传下载](http://blog.csdn.net/javaliuzhiyue/article/details/9357681)
> [Struts2利用stream直接输出Excel](http://chunpeng.iteye.com/blog/265222)
> [struts2输出并下载excel文件](http://blog.csdn.net/weinianjie1/article/details/5941042)
> [Struts2 +jquery+ajaxfileupload 实现无刷新上传文件](http://blog.csdn.net/make19830723/article/details/7055956)

#### 文件上传
在 jsp或者html 页面的文件上传表单里使用 file 标签. 如果需要一次上传多个文件, 就必须使用多个 file标签, 但它们的名字必须是相同的。表单中要设置`method`为`post`,`enctype`设置为`multipart/form-data`
```html
<form action="upload.action" method="post" enctype="multipart/form-data">
  <input type="file" id="file" name="upload">
  <br />
  <input type="submit">
</form>
```
或者搭配jQuery插件`ajaxfileupload`可以实现无刷新上传:
```html
<input type="file" name="upload">
<br />
<input type="button" value="upload" onclick="return upload();">
<script src="jquery.js"></script>  
<script src="ajaxfileupload.js"></script>  
<script>
function upload() {
  $.ajaxFileUpload({
    url:'upload.action',
    secureuri:false,
    fileElementId:'file', //文件上传域的ID
    dataType: 'json',
    success: function (data, status) {
      alert(status);
    },
    error: function (data, status, e) {
      alert(status);
    }
  });
}
</script>
```

在 Struts 应用程序里, FileUpload 拦截器和 Jakarta Commons FileUpload 组件可以完成文件的上传. 
```html
<interceptorname="fileUpload"class="org.apache.struts2.interceptor.FileUploadInterceptor"/>
```
该拦截器位于defaultStack中，每个Action访问都会执行

在 Action 中新添加 3 个和文件上传相关的属性. 这 3 个属性的名字必须是以下格式
如果是上传单个文件, `upload`属性的类型就是`java.io.File`, 它代表被上传的文件, 第二个和第三个属性的类型是`String`, 它们分别代表上传文件的**文件名**和**文件类型**
定义方式是分别是jsp页面file组件的名称+ContentType,jsp页面file组件的名称+FileName
如果上上传多个文件, 可以使用数组或 List
struts.xml
```xml
<action name="upload" class="com.action.UploadAction">  
    <interceptor-ref name="defaultStack">  
    <!--设置文件最大的限制（拦截器fileUpload的maximumSize字段）-->
        <param name="fileUpload.maximumSize">20000000</param>  
    <!--设置文件的要求的类型（拦截器fileUpload的allowedExtensions字段）-->  
        <param name="fileUpload.allowedExtensions">.jpg</param>  
    </interceptor-ref>  
    <!--出现错误后跳转的页面-->
    <result name="input">/upload.jsp</result>  
</action>  
<!--maximumSize与allowedExtensions在struts-default.xml文件中存在这样的一个拦截器，通过类名class可以找到类的源码，源码中就有这两个字段（不止两个，设置哪个用哪个）-->
<interceptor name="fileUpload" class="org.apache.struts2.interceptor.FileUploadInterceptor"/>
```
FileUpload 拦截器负责处理文件的上传操作, 它是默认的 defaultStack拦截器栈的一员. 
```java
public class UploadAction extends ActionSupport{
  // 属性名要和 input元素name值一致  
  private File upload;  
  //jsp页面file组件的名称+ContentType,  
  private String uploadContentType;  
  //jsp页面file组件的名称+FileName  
  private String uploadFileName;  
  //setter/getter
  
  public String execute() throws Exception {  
    //获得文件的真实路径创建文件，将接受的文件拷贝到这个文件中  
    String realPath = ServletActionContext.getServletContext().getRealPath("/upload");  
    File file = new File(realPath,uploadFileName);  
    FileUtils.copyFile(upload, file);  
    return NONE;  
  }
}
```
FileUpload 拦截器有 3 个属性可以设置.

* `maximumSize`: 上传文件的最大长度(以字节为单位), 默认值为 2 MB
* `allowedTypes`: 允许上传文件的类型, 各类型之间以逗号分隔
* `allowedExtensions`: 允许上传文件扩展名, 各扩展名之间以逗号分隔

若用户上传的文件大小大于给定的最大长度或其内容类型没有被列在 `allowedTypes`,`allowedExtensions`参数里, 将会显示一条出错消息. 与文件上传有关的出错消息在`struts-messages.properties`文件里预定义了这些字段错误后给出的信息.(org.apache.struts2包下)，可以在文件上传 Action相对应的资源文件中重新定义错误消息（国际化文件来覆盖原本的英文提示内容）, 但需要在 struts.xml文件中配置使用。

**注：在jsp中回显错误信息用 `<s:fielderror/>`显示错误信息**

修改显示错误的资源文件的信息
第一步:创建新的资源文件 例如`fileuploadmessage.properties`,放置在src下在该资源文件中增加如下信息
```
struts.messages.error.uploading=上传错误: {0}
struts.messages.error.file.too.large=上传文件太大: {0} "{1}" "{2}" {3}
struts.messages.error.content.type.not.allowed=上传文件的类型不允许: {0} "{1}" "{2}" {3}
struts.messages.error.file.extension.not.allowed=上传文件的后缀名不允许: {0} "{1}" "{2}" {3}
# 备注：{0}:<inputtype=“file” name=“uploadImage”>中name属性的值
#      {1}:上传文件的真实名称
#      {2}:上传文件保存到临时目录的名称
#      {3}:上传文件的类型(对struts.messages.error.file.too.larg是上传文件的大小)
```
第二步:在struts.xml文件加载该资源文件
```xml
<!-- 配置上传文件的出错信息的资源文件 -->
<constantname="struts.custom.i18n.resources" value=“配置文件基名“/> 
```

#### 多文件上传
客户端可以使用多个`<input type="file">`同时进行文件上传
如果`input filename`是不同的，则要配置多组文件、文件名、类型字段。 

```java
public class UploadAction extends ActionSupport{
  // 属性名要和 input元素name值一致  
  private File[] upload;  
  private String[] uploadContentType;  
  private String[] uploadFileName;  
  //setter/getter
  
  public String execute() throws Exception {  
    //获得文件的真实路径创建文件，将接受的文件拷贝到这个文件中  
    String realPath = ServletActionContext.getServletContext().getRealPath("/upload");  
    for (int i = 0; i < uploadFileName.length; i++) {  
        File file = new File(realPath,uploadFileName[i]);  
        FileUtils.copyFile(upload[i], file);              
    }
    return NONE;  
  }
}
```

#### 文件下载
struts2提供了**stream**结果类型，该结果类型就是专门用于支持文件下载功能的
指定stream结果类型 需要指定一个 `inpuName`参数，该参数指定一个输入流，提供被下载文件的入口
在struts-default.xml文件中，结果集定义了一种**stream**类型
```xml
<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>
```
HTTP响应 可以以一个流方式发送到客户端
注：请求中文文件名下载，get方式提交，需要手动编码`new String(filename.getBytes("ISO-8859-1"),"utf-8");`文件下载所必须的三点

两个协议头信息ContentType(类型)：文件类型
ContentDisposition(下载附件名称)：attachment;filename=文件名
一个文件下载流struts.xml文件中的配置
```xml
<action name="download" class="com.action.DownloadAction">  
  <!-- 文件下载 使用流 结果集 -->  
  <result type="stream">  
    <!-- 文件MIME类型 -->  
    <!-- ${contentType} OGNL写法，用于读取Action中的getContentType方法，动态获取文件类型，下载什么类型就是什么类型  -->  
    <param name="contentType">${contentType}</param>  
    <!-- 下载附件名称contentDisposition 格式为：attachment;filename=xxxx ${filename}用于读取Action中的getFilename方法，动态获取文件名-->  
    <param name="contentDisposition">attachment;filename=${filename}</param>  
    <!-- 设置流方法名称,必须Action内部提供getTarget返回值InputStream ，用来返回文件内容  -->  
    <param name="inputName">target</param>  
  </result>  
</action>
```
action
```java
public class DownloadAction extends ActionSupport {
  //文件名的成员变量，用于接受传递过来的文件名参数
  private String filename;
  public void setFilename(String filename) throws UnsupportedEncodingException {
    //手动编解码，防止乱码
    if(filename!=null){
      this.filename = new String(filename.getBytes("ISO8859-1"),"UTF-8");
    }
  }
  
  public String execute() throws Exception {
    return SUCCESS;
  }
  
  //提供getTarget放回InputStream流
  public InputStream getTarget() throws FileNotFoundException{
    String realPath = ServletActionContext.getServletContext().getRealPath("/upload");
    File downfile = new File(realPath,filename);
    return new FileInputStream(downfile);
  }
  
  public String getConyentType(){
    //通过ServletContext的getMimeType方法获取MIME类型
    String mimeType = ServletActionContext.getServletContext().getMimeType(filename);
    return mimeType;
  }
  
  public String getFilename() throws UnsupportedEncodingException {
    return URLEncoder.encode(filename,"UTF-8"); 
    }
  }
}
```
前端页面
```html
<a href="${pageContext.request.contextPath }/download.action?filename=abc.jpg">abc</a>
```


----------

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