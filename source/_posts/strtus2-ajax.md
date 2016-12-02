title: "struts2使用ajax技术返回字符串"
date: 2015-04-21 00:15:45
tags: "ajax"
categories: ["java", "struts2"]
---

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
