title: "Servlet文件上传与下载"
date: 2015-04-24 10:00:55
tags: ["servlet", "upload"]
categories: "java"
---

#### 文件上传
##### 使用apache commons 

> [利用apache的fileupload组件实现文件上传](http://www.java3z.com/cwbwebhome/article/article5/51047.html)
> [Apache Commons fileUpload实现文件上传](http://zhangjunhd.blog.51cto.com/113473/18331)

html表单需要设置`post`方法及`enctype`为`multipart/form-data`
```html
<form name="form1" action="fileUpload" method="post"
       enctype="multipart/form-data">
  <input type="file" name="myfile"><br>
  <input type="submit" name="submit" value="提交">
</form>
```
对应的servlet
```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	//临时存放目录
	String tempdir =this.getServletContext().getRealPath("/tmp");
	File tempFile = new File(tempdir);
	if(!tempFile.exists()){
		tempFile.mkdirs();
	}
	//存放目录
    String savedir =this.getServletContext().getRealPath("/upload");
    File saveFile = new File(savedir);
    if(!saveFile.exists()){
    	saveFile.mkdirs();
    }
	//检查表弟enctype属性是否为multipart/form-data
	if(ServletFileUpload.isMultipartContent(request)){
		request.setCharacterEncoding("UTF-8");
		DiskFileItemFactory factory = new DiskFileItemFactory();
		//内存最大占用
		factory.setSizeThreshold(1024000);
		//设置缓冲区目录
		factory.setRepository(new File(tempdir));
		ServletFileUpload upload = new ServletFileUpload(factory);
		//单个文件最大值byte
		upload.setFileSizeMax(102400000);
		//所有上传文件的总和最大值byte
		upload.setSizeMax(204800000);
		List<FileItem> items = null;
        try {
			items = upload.parseRequest(request);
		} catch (FileUploadException e) {
			e.printStackTrace();
		}
        Iterator<FileItem> it = items.iterator();
        while(it.hasNext()){
        	FileItem fileItem = (FileItem) it.next();
        	//如果是普通字段
        	if(fileItem.isFormField()){
        		System.out.println(fileItem.getFieldName());	//获取item的name值
        		System.out.println(fileItem.getString("UTF-8"));//获取item的value值
        	}else{
        		System.out.println(fileItem.getName());	//获取文件名
        		System.out.println(fileItem.getContentType());  //获取文件类型
        		System.out.println(fileItem.getSize());  //获取文件大小
        		if(fileItem.getName()!=null && fileItem.getSize()>0){
        			File file = new File(savedir, fileItem.getName());
        			try {
						fileItem.write(file);
					} catch (Exception e) {
						e.printStackTrace();
					}
        		}
        	}
        }
        
	}
}
```

----

##### 使用Servlet3.0文件上传接口

> [Servlet 3.0新特性——文件上传接口](http://pisces-java.iteye.com/blog/723125)
> [Servlet 3.0笔记之超方便的文件上传支持](http://www.blogjava.net/yongboy/archive/2011/01/15/346202.html)

页面与上文一样，表单需要设置方法类型为`post`且`enctype`为`multipart/form-data`。
在Servlet中添加`@MultipartConfig`注解，如下所示
```java
@MultipartConfig(
  location="D:/",//文件存放路径
  maxFileSize=1024*1024*1024, //上传文件最大值(单位:字节)
  fileSizeThreshold=10*1024*1024 //当数据量大于该值时，内容将被写入文件
  //maxRequestSize=8*1024*1024 针对该 multipart/form-data 请求的最大数量，默认值为 -1，表示没有限制
)
```
接下来是`doPost`方法
```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	request.setCharacterEncoding("UTF-8");
	//多文件上传
	Collection<Part> parts = request.getParts();
	//单文件上传
	//Part part = request.getPart("file");
	//遍历所有的表单内容，将表单中的文件写入上传文件目录
	for (Iterator<Part> iterator = parts.iterator(); iterator.hasNext();) {  
        Part part = iterator.next();  
        //从Part的content-disposition中提取上传文件的文件名  
        //获取header信息中的content-disposition，如果为文件，则可以从其中提取出文件名  
        String fileName = part.getHeader("content-disposition");
        if(fileName!=null){  
            part.write(fileName);  
        }  
    }
}
```

----

#### 文件下载
> [servlet实现文件下载](http://blog.sina.com.cn/s/blog_49a269ae01008yv2.html)

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	String fileName = "test.txt";
	response.setContentType("text/plain");
	response.setHeader("Location", fileName);
	response.setHeader("Content-Desposition", "attachment; filename="+fileName);
	OutputStream os = response.getOutputStream();
	InputStream is = new FileInputStream(fileName);
	byte[] buffer = new byte[1024];
	int len;
	while((len = is.read(buffer))!=-1){
		os.write(buffer, 0, len);
	}
	os.flush();
	os.close();
}
```


