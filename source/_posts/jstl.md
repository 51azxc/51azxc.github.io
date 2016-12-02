title: "jstl的相关标签"
date: 2015-04-03 23:42:12
tags: ["jstl", "jsp"]
categories: "java"
---

### 迭代标签

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


### 判断标签

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

### 使用fmt标签显示小数

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

### fn函数

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