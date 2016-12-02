title: "struts2中 # % $ 的区别和用法"
date: 2015-04-21 00:10:02
tags: "ognl"
categories: ["java", "struts2"]
---

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
