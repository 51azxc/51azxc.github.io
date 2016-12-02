title: "jquery部分知识点"
date: 2015-05-11 17:29:10
tags: ["ajax", "date", "selector", "css"]
categories: ["javascript", "jQuery"]
---

### 手风琴特效
> [Making accordion menu using jquery](http://roshanbh.com.np/2008/06/accordion-menu-using-jquery.html)

html
```html
<div id="firstpane" class="menu_list">
  <p class="menu_head">Header-1</p>
    <div class="menu_body">
        <a href="#">Link-1</a>
    </div>
  <p class="menu_head">Header-2</p>
    <div class="menu_body">
        <a href="#">Link-1</a>
    </div>
  <p class="menu_head">Header-3</p>
    <div class="menu_body">
        <a href="#">Link-1</a>
   </div>
</div>
```
css
```css
.menu_list {
	width: 150px;
}
.menu_head {
	padding: 5px 10px;
	cursor: pointer;
	position: relative;
	margin:1px;
       font-weight:bold;
}
.menu_body {
	display:none;
}
.menu_body a {
  display:block;
  color:#006699;
  background-color:#EFEFEF;
  padding-left:10px;
  font-weight:bold;
  text-decoration:none;
}
.menu_body a:hover {
  color: #000000;
  text-decoration:underline;
}
```
js(需要引入jquery文件)
```js
//点击事件，如果需要移动鼠标出效果则是mouseover方法
$("#firstpane p.menu_head").click(function()
{
    $(this).next("div.menu_body").slideToggle(300).siblings("div.menu_body").slideUp("slow");
});
```

----------

### jquery跳出each循环

> [Jquery跳出each循环](http://www.cnblogs.com/abllyboy/archive/2011/03/11/1981437.html)

jquery each循环中，`return false`则是`break`功能，`return true`则是`continue`功能

----------

### 判断`div`是否隐藏
> 来自 [jquery判断div是否隐藏实例](http://blog.csdn.net/ddxkjddx/article/details/5766603)

```js
$("#div_id").is(":visible");//是否显示
$("#div_id").is(":hidden");//是否隐藏
```

----------

### 选择非只读的控件
> 来自 [jQuery “not readonly” selector](http://stackoverflow.com/questions/3708764/jquery-not-readonly-selector)

```js
$("input[type=text]:not([readonly])")
```

----------

### 比较日期大小
> 来自 [【jQuery日期处理】两个时间大小的比较](http://blog.csdn.net/sxdtzhaoxinguo/article/details/11291291)

```js
function checkDate(val) {
	var d1 = new Date(val);
	var d2 = new Date();
	if (d1 < d2) {
		return true;
	} else {
		return false;
	}
}
```

----------

### jquery post方法提交json数据不能被解析

> [使用jQuery POST提交数据返回的JSON是字符串不能解析为JSON对象](http://blog.csdn.net/pzp_118/article/details/8423925)

`$.post`方法最后一个参数指定返回数据类型，写明json即可
```
$.post(url,data,function(){},'json');
```

### jquery提交post数据在firefox正常在chrome和ie下乱码解决
> 来自 [jquery提交post数据在firefox正常在chrome和ie下乱码解决](http://blog.i5a6.com/973.html)

1.使用`$.ajaxSetup`指定字符集为"UTF-8"
```
$.ajaxSetup({
  contentType: "application/x-www-form-urlencoded; charset=utf-8"
});
$.post(url,data,function(){},'json');
```
2.在`$.ajax`方法中使用`contentType`指定字符集
```
$.ajax({
  url:url,
  type:"POST",
  data:data,
  contentType:"application/x-www-form-urlencoded; charset=utf-8",
  dataType:"json",
  success: function(){}
})
```

----------

### JQuery ajax向后台传递数组

> [JQuery Ajax向后台传递数组](http://empirel.iteye.com/blog/1763010)

前端传送是，最好指定参数名带数组的标识`[]`
```
$.ajax({
  url:url,
  type:"POST",
  data:{"aa[]":new Array}
  success: function(){}
})
```
后台接收时，使用`getParameterValues`方法即可
```
String[] aa = request.getParameterValues();
```

----------

### jQuery中`prop()`方法和`attr()`方法的区别

> [jQuery中prop()方法和attr()方法的区别](http://lemmychrist.blog.163.com/blog/static/98732963201391485225489/)

`prop()`方法是jQuery1.6以后出现的方法，与`attr()`方法接近，都是对组件自身一些属性操作的方法，对于`async`,`autofocus`,`checked`,`location`,`multiple`,`readOnly`,`selected`这些属性可以使用`prop()`方法，其余的建议用`attr()`方法

```
<input id="c1" type="checkbox" checked>
<input id="c2" type="checkbox">
<script>
$(function(){
  //获取选中状态
  $("#c1").prop("checked"); //返回true
  $("#c2").prop("checked"); //返回false
  $("#c1").attr("checked"); //返回checked
  $("#c2").attr("checked"); //返回undefined
  
  //对选中进行操作
  $("#c1").prop("checked",true); //取消用false
  $("#c1").attr("checked","true");
  //取消选中
  $("#c1").removeAttr("checked");
});
</script>
```
对于html5新属性`data-*`，如果使用`prop`则可以取值但无法赋值，`attr`则可以赋值
```
<input type="text" id="t1" data-test="t">
<script>
$(function(){
  //取值
  $("#t1").prop("data-test"); //返回t
  $("#t1").attr("data-test"); //返回t
  $("#t1").data("test"); //返回t
  
  //赋值
  $("#c1").attr("data-test","tt"); 
});
</script>
```

----------

### jQuery,javascript获得网页的高度和宽度

> [jQuery,javascript获得网页的高度和宽度](http://blog.csdn.net/ljw520204/article/details/6925775)

**javascript**
网页可见区域宽： `document.body.clientWidth`
网页可见区域高： `document.body.clientHeight`
网页可见区域宽： `document.body.offsetWidth` (包括边线的宽)
网页可见区域高： `document.body.offsetHeight` (包括边线的高)
网页正文全文宽： `document.body.scrollWidth`
网页正文全文高： `document.body.scrollHeight`
网页被卷去的高： `document.body.scrollTop`
网页被卷去的左： `document.body.scrollLeft`
网页正文部分上： `window.screenTop`
网页正文部分左： `window.screenLeft`
屏幕分辨率的高： `window.screen.height`
屏幕分辨率的宽： `window.screen.width`
屏幕可用工作区高度： `window.screen.availHeight`
屏幕可用工作区宽度： `window.screen.availWidth`

**jquery**
获取浏览器显示区域的高度 ： `$(window).height()`;
获取浏览器显示区域的宽度 ：`$(window).width()`; 
获取页面的文档高度 ：`$(document).height()`; 
获取页面的文档宽度 ：`$(document).width()`;
获取滚动条到顶部的垂直高度 ：`$(document).scrollTop()`; 
获取滚动条到左边的垂直宽度 ：`$(document).scrollLeft()`; 

计算元素位置和偏移量 
`offset`方法是一个很有用的方法，它返回包装集中第一个元素的偏移信息。默认情况下是相对body的偏移信息。结果包含 top和left两个属性。 
`offset(options, results) `
`options.relativeTo`　　指定相对计算偏移位置的祖先元素。这个元素应该是relative或absolute定位。省略则相对body。 
`options.scroll`　　是否把 滚动条计算在内，默认TRUE 
`options.padding`　　是否把padding计算在内，默认false 
`options.margin` 　　是否把margin计算在内，默认true 
`options.border`　　是否把边框计算在内，默认true 

----------

### jQuery元素追加和删除

> [Jquery元素追加和删除](http://www.cnblogs.com/william-lin/archive/2012/08/12/2635402.html)

**追加元素**
`append()`: 向每个匹配的元素内追加内容
```
<ul></ul>

$("ul").append("<li>AA</li>");

<ul>
  <li>AA</li>
</ul>
```
`appendTo()`: 该方法和append()相反
```
<ul></ul>

$("<li>AA</li>").appendTo ("ul");

<ul>
  <li>AA</li>
</ul>
```
`prepend()`:向每个匹配的元素内部前置内容
```
<p>aa</p>

$("p").prepend("<b>bb</b>");

<p><b>bb</b>aa</p>
```
`prependTo()`:该方法和prepend()相反
```
<p>aa</p>

$(<b>bb</b>").prependTo("p");

<p><b>bb</b>aa</p>
```
`after()`:在每个匹配的元素**之后**插入内容
```
<p>aa</p>

$("p").after("<b>bb</b>");

<p>aa</p><b>bb</b>
```
`insertAfter()`:该方法和after()相反
```
<p>aa</p>

$(<b>bb</b>").insertAfter("p");

<p>aa</p><b>bb</b>
```
`before()`:在每个匹配的元素**之前**插入内容
```
<p>aa</p>

$("p").before("<b>bb</b>");

<b>bb</b><p>aa</p>
```
`insertBefore()`:该方法和before()相反
```
<p>aa</p>

$(<b>bb</b>").insertBefore("p");

<b>bb</b><p>aa</p>
```

**删除元素**
`remove()`:删除元素
`empty()`:清空元素文本内容，元素本身还存在

----------

### jquery选择器

> [从零开始学习jQuery (二) 万能的选择器](http://www.cnblogs.com/zhangziqiu/archive/2009/05/03/jQuery-Learn-2.html)

**1. 基础选择器 Basics**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| #id | 根据元素id选择 | `$("#tid")` 选择ID为tid的元素 |
| element | 根据元素的名称选择 | `$("a")` 选择所有`<a>`元素 |
| `.class` | 根据元素的css类选择 | `$(".imgClass")`选择所用CSS类为imgClass的元素 |
| `*` | 选择所有元素 | `$("*")`选择页面所有元素 |
| selector1, selector2 | 可以将几个选择器用","分隔开然后再拼成一个选择器字符串.会同时选中这几个选择器匹配的内容. | `$("#tid, a, .imgClass")` |


**2.层次选择器 Hierarchy**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| ancestor descendant | 使用"form input"的形式选中form中的所有input元素.即ancestor(祖先)为from, descendant(子孙)为input. | `$("#tid input")` 选择ID为tid的元素中所有的input元素 |
| parent > child | 选择parent的直接子节点child.  child必须包含在parent中并且父类是parent元素. | `$(".myList>li")`选择CSS类为myList元素中的直接子节点`<li>`对象 |
| prev + next | prev和next是两个同级别的元素. 选中在prev元素后面的next元素 | `$("#tid+img")`选在id为hibiscus元素后面的img对象 |
| prev ~ siblings | 选择prev后面的根据siblings过滤的元素.siblings是过滤器 | `$("#someDiv~[title]")`选择id为someDiv的对象后面所有带有title属性的元素 |

**3.基本过滤器 Basic Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `:first` | 匹配找到的第一个元素 | 查找表格的第一行:`$("tr:first")` |
| `:last` | 匹配找到的最后一个元素 | 查找表格的最后一行:`$("tr:last")` |
| `not(selector)` | 去除所有与给定选择器匹配的元素 | 查找所有未选中的 input 元素: `$("input:not(:checked)")` |
| `:even` | 匹配所有索引值为偶数的元素，从 0 开始计数 | 查找表格的偶数行:`$("tr:even")` |
| `:odd` | 匹配所有索引值为奇数的元素，从 0 开始计数 | 查找表格的奇数行:`$("tr:odd")` |
| `:eq(index)` | 匹配一个给定索引值的元素,index从 0 开始计数 | 查找第二行:`$("tr:eq(1)")` |
| `:gt(index)` | 匹配所有大于给定索引值的元素,index从 0 开始计数 | 查找第二第三行，即索引值是1和2，也就是比0大:`$("tr:gt(0)")` |
| `:lt(index)` | 匹配所有小于给定索引值的元素,index从 0 开始计数  | 查找第一第二行，即索引值是0和1，也就是比2小:`$("tr:lt(2)")` |
| `:header` | 选择所有h1,h2,h3一类的header标签 | 给页面内所有标题加上背景色: `$(":header").css("background", "#EEE");` |
| `:animated` | 匹配所有正在执行动画效果的元素 | 查找不执行动画效果的div元素: `$("div:not(:animated)") ` |

**4. 内容过滤器 Content Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `:contains(text)` | 匹配包含给定文本的元素 | 查找所有包含 "John" 的 div 元素:`$("div:contains('John')")` |
| `:empty` | 匹配所有不包含子元素或者文本的空元素 | 查找所有不包含子元素或者文本的空元素:`$("td:empty")` |
| `:has(selector)` | 匹配含有选择器所匹配的元素的元素 | 给所有包含 p 元素的 div 元素添加一个 text 类: `$("div:has(p)").addClass("test");` |
| `:parent` | 匹配含有子元素或者文本的元素 | 查找所有含有子元素或者文本的 td 元素:`$("td:parent")` |

**5.可见性过滤器  Visibility Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `:hidden` | 匹配所有的不可见元素 | 查找所有不可见的 tr 元素:`$("tr:hidden")` |
| `:visible` | 匹配所有的可见元素 | 查找所有可见的 tr 元素:`$("tr:visible")` |

**6.属性过滤器 Attribute Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `[attribute]` | 匹配包含给定属性的元素 | 查找所有含有 id 属性的 div 元素: `$("div[id]")` |
| `[attribute=value]` | 匹配给定的属性是某个特定值的元素 | 查找所有 name 属性是 a 的 input 元素: `$("input[name='a']")` |
| `[attribute!=value]` | 匹配给定的属性不是某个特定值的元素 | 查找所有 name 属性不是 a 的 input 元素: `$("input[name!='a']")` |
| `[attribute^=value]` | 匹配给定的属性是以某些值开始的元素 | 查找所有 name 属性是以 a 开头的 input 元素 `$("input[name^='a']")` |
| `[attribute$=value]` | 匹配给定的属性是以某些值结尾的元素 | 查找所有 name 属性是以 a 结尾的 input 元素 `$("input[name$='a']")` |
| `[attribute*=value]` | 匹配给定的属性是以包含某些值的元素 | 查找所有 name 属性是包含 a 的 input 元素 `$("input[name*='a']")` |
| [attributeFilter1][attributeFilter2] | 复合属性选择器，需要同时满足多个条件时使用。 | 找到所有含有 id 属性，并且它的 name 属性是以 man 结尾的:`$("input[id][name$='man']")` |

**7.子元素过滤器 Child Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| :nth-child(index/even/odd/equation) | 匹配其父元素下的第N个子或奇偶元素 | 在每个 ul 查找第 2 个li: `$("ul li:nth-child(2)")` |
| `:first-child` | 匹配第一个子元素 | 在每个 ul 中查找第一个 li: `$("ul li:first-child"`) |
| `:last-child` | 匹配最后一个子元素 | 在每个 ul 中查找最后一个 li: `$("ul li:last-child")` |
| `:only-child` | 如果某个元素是父元素中唯一的子元素，那将会被匹配。如果父元素中含有其他元素，那将不会被匹配。 | 在 ul 中查找是唯一子元素的 li:`$("ul li:only-child")` |

**8.表单选择器 Forms**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `:input` | 匹配所有 input, textarea, select 和 button 元素 | `$(":input")` |
| `:text` | 匹配所有的文本框 | `$(":text")` |
| `:password` | 匹配所有密码框 | `$(":password")` |
| `:radio` | 匹配所有单选按钮 | `$(":radio")` |
| `:checkbox` | 匹配所有的复选按钮 | `$(":checkbox")` |
| `:submit` | 匹配所有提交按钮 | `$(":submit")` |
| `:image` | 匹配所有图像域 | `$(":image")` |
| `:reset` | 匹配所有重置按钮 | `$(":reset")` |
| `:button` | 匹配所有按钮 | `$(":button")` |
| `:file` | 匹配所有文件域 | `$(":file")` |

**9.表单过滤器 Form Filters**

| 名称 | 说明 | 举例 |
| --- | --- | --- |
| `:enabled` | 匹配所有可用元素 | 查找所有可用的input元素: `$("input:enabled")` |
| `:disabled` | 匹配所有不可用元素 | 查找所有不可用的input元素:`$("input:disabled")` |
| `:checked` | 匹配所有选中的被选中元素(复选框、单选框等，不包括select中的option) | 查找所有选中的复选框元素:`$("input:checked")` |
| `:selected` | 匹配所有选中的option元素 | 查找所有选中的选项元素:`$("select option:selected")` |


----

### jQuery事件
#### jquery使用on函数绑定hover事件

> [Is it possible to use jQuery .on and hover?](http://stackoverflow.com/questions/9827095/is-it-possible-to-use-jquery-on-and-hover)
> [jQuery 使用 .on( ) 无法绑定 hover，jQuery 2.0 如何给生成的内容绑定 hover？](http://segmentfault.com/q/1010000000319439)

jQuery的`.on()`方法不能绑定`hover`事件，配合`mouseenter`及`mouseleave`事件可达到效果
```
$(".selector").on({
    mouseenter: function () {
        //stuff to do on mouse enter
    },
    mouseleave: function () {
        //stuff to do on mouse leave
    }
});
```

----------

#### jquery回车键事件

> [基于jquery的button默认enter事件(回车事件)。]http://www.jb51.net/article/27170.htm)
> [jquery中如何实现按回车触发按钮事件](http://blog.csdn.net/wym3587/article/details/6938292)

按回车键触发按钮点击事件
```
$("body").keydown(function() {
  if (event.keyCode == "13") {//keyCode=13是回车键
    $('#btnSumit').click();
  }
});
```

----------

#### 按tab键跳过readonly的输入框

> [怎样实现按TAB键后跳过readonly的文本框](http://bbs.csdn.net/topics/340016321)

给`readonly`属性的输入框指定`tabindex`属性为 -1 即可实现按tab键时跳过readonly的输入框
```
$("input[type=text][readonly]").attr("tabindex",-1);
```
