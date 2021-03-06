title: "Java知识点收集"
date: 2015-04-11 16:11:03
tags: ["Java", "tomcat", "jdbc", "servlet", "filter", "jsp"]
categories: "Java"
---

收集平时碰到的一些小问题的解决方法。

<!-- more -->

### java.lang.ClassCastException: oracle.sql.BLOB

> [奇怪的错误 java.lang.ClassCastException: oracle.sql.BLOB](http://www.haodaima.net/art/783483)

tommcat目录下的lib文件夹中与项目中WEB-INF/lib下有同样的包导致，去掉其中一个即可。

----

###  java.sql.SQLException: 对只转发结果集的无效操作

> [java.sql.SQLException: 对只转发结果集的无效操作](http://zhidao.baidu.com/question/38976693.html)

`stat = conn.createStatement();`改为 `stmt=conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,ResultSet.CONCUR_READ_ONLY);`就可以了
 
分析: 异常出现于移动结果集的指针时,原因是在生成statement对象的时候提供的参数不同
无参数的那个方法使用的是默认参数,`statement`执行后得到的结果集类型为 `ResultSet.TYPE_FORWARD_ONLY`.这种类型的结果集只能通过`rs.next();`方法逐条读取,使用其他方法就会报异常. 如果想执行一些复杂的移动结果集指针的操作就要使用其他参数了
顺便简单介绍一下各个参数:

   1. `ResultSet.TYPE_FORWARD_ONLY`   (略)
   2. `ResultSet.TYPE_SCROLL_INSENSITIVE`  双向滚动，但不及时更新，就是如果数据库里的数据修改过，并不在ResultSet中反应出来。
   3. `ResultSet.TYPE_SCROLL_SENSITIVE`  双向滚动，并及时跟踪数据库里的更新,以便更改ResultSet中的数据。
   4. `ResultSet.CONCUR_READ_ONLY`  只读取ResultSet
   5. `ResultSet.CONCUR_UPDATABLE`  用ResultSet更新数据库

----

###  bean user not found within scope

> [bean user not found within scope](http://bbs.csdn.net/topics/290064466)

如果是
```html
<jsp:useBean id="person" type="bean.Bean_person" scope="request"/>
```
把他改成
```html
<jsp:useBean id="person" class="bean.Bean_person" scope="request"/>
```

----

### PrintWriter 中文乱码

> [关于Servlet的PrintWriter 中文乱码问题](http://www.cnblogs.com/mohe/p/3287306.html)

指定字符集为`UTF-8`即可
```java
response.setCharacterEncoding("UTF-8");
response.getWriter().write("");
```

----

### 配置filter拦截路径

> [请问为什么这个filter配置不对呢](http://www.iteye.com/problems/40229)

Servlet JSR（2.3） 11.2章在web应用描述文件中，匹配的定义如下：
以'/'开始，以'/* '为结尾的字符串,用作路径匹配
以'*.'开始的字符串，用作扩展名匹配
包含'/'字符串，定义一个默认的servlet。如匹配的servlet路径是请求URI路径的最小上下文路径，路经的info为空。
其它的字符串用作精确的匹配。

如果要拦截"/page/* .jsp",应该配置"/page/* ",然后再到filter中判断后缀是否为"jsp"

----

### "`<%@ include file="" %>` ，`<jsp:include page="">` 与 `<c:import url="" />` 的应用及区别"

> [`<%@ include file="" %>` ，`<jsp:include page="">` 与 `<c:import url="" />` 的应用及区别](http://jackroomage.iteye.com/blog/1868358)

* `<%@ include file="" %>`
伪指令在某些网站上有其用武之地。例如，如果站点包含一些（如果有变化，也很少）几乎没有变化的页眉、页脚和导航文件，那么基本的 include 伪指令是这些组件的最佳选项。由于 include 伪指令采用了高速缓存，因此只需放入包含文件一次，其内容就会被高速缓存，其结果会是极大地提高了站点的性能。

* `<jsp:include page="">` 
flush 指示在读入包含内容之前是否清空任何现有的缓冲区。JSP 1.1 中需要 flush 属性，因此，如果代码中不用它，会得到一个错误。但是，在 JSP 1.2 中， flush 属性缺省为 false
设置flush为true，就是说，如果你的缓冲区的内容很多了，就将数据读出，以免数据泄漏，造成错误。服务器端页面缓冲，大致的意思是，在将生成的HTML代码送到客户端前，先在服务器端内存中保留，因为解释JSP或Servlet变成HTML是一步步进行的，可以在服务器端生成完HTML或生成一部分HTML（所占用字节数已达到指定的缓冲字节数）后再送到客户。如果不缓冲，就会解释生成一句HTML就向客户端送一句。
你还可以用<jsp:param>传递一个或多个参数给JSP 网页。
```html
<jsp:include page="header.jsp" flush="true">
 <jsp:param name="pageTitle" value="newInstance.com"/>
 <jsp:param name="pageSlogan" value=" " />
</jsp:include>
```

* `<c:import url="" />`
`<c:import>` 标签的功能类似于`<jsp:include >`,但是相比来说还是`<c:import>`标签的功能更强大一些。`<jsp:include >`标签只能导入同一个web容器内的资源，而`<c:import>`除此之外也可以导入外网的资源。同时`<c:import>`也可以把页面内容插入到页面中，还可以把内容保存到String对象中。例如：
```html
<c:import url="http://www.baidu.com" var="urlStr" />
<c:out value="${urlStr}" />
```

----

### 对`request.getSession`理解

> [关于request.getSession(true/false/null)的区别](http://blog.csdn.net/gaolinwu/article/details/7285783)
> [对request.getSession(false)的理解（附程序员常疏忽的一个漏洞）](http://blog.csdn.net/xxd851116/article/details/4296866)

**`getSession(boolean create)`**意思是返回当前reqeust中的HttpSession ，如果当前reqeust中的HttpSession 为null，当create为true，就创建一个新的Session，否则返回null；

简而言之：

`HttpServletRequest.getSession(ture)` 等同于 **HttpServletRequest.getSession()**;
`HttpServletRequest.getSession(false)` 等同于**如果当前Session没有就为null**；

当向Session中存取登录信息时，一般建议：`HttpSession session =request.getSession()`;
当从Session中获取登录信息时，一般建议：`HttpSession session =request.getSession(false)`;

----

### jstl的相关标签

#### 迭代标签

> [jstl中<c:forEach>的用法](http://blog.csdn.net/honey_claire/article/details/7664165)
> [C:forEach 使用方法](http://www.cnblogs.com/qingyuanintel/archive/2012/11/29/2794154.html)

jstl中的`<c:forEach>`标签用于迭代需要的数组。
使用jstl标签都需要在页面中引用jstl标签库

```html
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

`<c:forEach>`的语法定义如下:

```html
<c:forEach var="item" items="${list}" varStatus="i" begin="1" end="10" step="1">
  ${item}
</c:forEach>
```

标签有如下属性:

- var: 需要迭代的参数名称
- items: 需迭代的集合
- varStatus: 迭代变量的名称，可以通过它来访问自身的信息
- begin: 迭代开始的位置
- end: 迭代结束的位置
- step: 迭代的步长。

`varStatus`包含了一系列的特性，它们描述了当前的迭代状态，主要有

- current: 迭代至当前集合众的项
- index: 当前的迭代索引
- count: 集合的长度
- first: 判断是否为集合第一个元素，返回类型为boolean
- last: 判断是否为集合最后一个元素，返回类型为boolean
- begin: 获取迭代开始的位置
- end: 获取迭代结束的位置
- step: 获取迭代步长

----


#### 判断标签

> [JSTL 的 if else : 有 c:if 没有 else 的处理](http://blog.csdn.net/xiyuan1999/article/details/4412009)
> [用jstl的if或when标签判断字符串是否为空](http://tianhandigeng.iteye.com/blog/938253)
> [JSTL: empty 可以减少很多繁冗的判空](http://blog.csdn.net/queenjade/article/details/7444059)

在jstl中判断可以使用`<c:if test=""></c:if>`标签来进行判断，但是却没有`<c:else>`这样的分支，如果要达到这种要求，可以使用`<c:choose>`标签

```html
<c:choose>
  <c:when test="">
  </c:when>
  ···
  <c:otherwise>
  </c:otherwise>
</c:choose>
```

在以上代码中，当`<c:when>`分支的条件都不符合时，则会进入`<c:otherwise>`分支。

当判断字符串是否为空时，除了可以使用`<c:if test="${str != ''}">`之外，还可以使用`empty`关键字来判断，如`<c:if test="${! empty str}">`.不仅仅如此，`empty`还可以用来判断集合是否为空等。

----

#### 使用fmt标签显示小数

需要另外引入标签库

```html
<%@tagliburi="http://java.sun.com/jsp/jstl/fmt"prefix="fmt"%>
```

`<fmt:formatNumber>`标签可以用来格式化需要的数字，例如如果需要输出的数字为两位小数，则可以使用它

```html
<c:set var="balance" value="112.345" />
<fmt:formatNumber type="number" maxFractionDigits="2" value="${balance}" />
```

`maxFractionDigits`属性表示小数点后最大的位数，关于`formatNumber`的其他参数可以参考[这里][1].

[1]: http://www.w3cschool.cc/jsp/jstl-format-formatnumber-tag.html

----

#### fn函数

[JSTL（fn函数）](http://blog.csdn.net/donghustone/article/details/6711999)

首先引入函数

```html
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

`fn`主要有如下函数

| 方法        | 说明   |
| --------   | :-----  |
| fn:contains(string, substring) | 判断string是否包含substring |
| fn:containsIgnoreCase(string, substring) | 判断string是否包含substring(不计大小写) |
| fn:endsWith(string, suffix) | 判断string是否以suffix结尾 |
| fn:escapeXml(string) | 跳过可以作为XML标记的字符 |
| fn:indexOf(string, substring) | 返回substring第一次在string出现的位置 |
| fn:join(array, separator) | 形成一个字符串以array+separator组成 |
| fn:length(item) | 返回item的长度 String/Collection等 |
| fn:replace(string, before, after) | 在string中用after替换掉before字符串 |
| fn:split(string, separator) | 将string通过separator分割成数组 |
| fn:substring(string, begin, end) | 通过开始位置begin与结束位置end从string截取子字符串 |
| fn:substringAfter(string, substring) | 返回字符串在指定子串之后的子集 |
| fn:substringBefore(string, substring) | 返回字符串在指定子串之前的子集 |
| fn:toLowerCase(string) | 转换为小写 |
| fn:toUpperCase(string) | 转换为大写 |
| fn:trim(string) | 去除首位空格 |

具体示例可以查看[JSTL函数][2]

[2]: http://www.w3cschool.cc/jsp/jsp-jstl.html
