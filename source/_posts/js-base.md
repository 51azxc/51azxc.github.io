title: "javascript 填坑"
date: 2015-05-11 14:17:37
tags: ["callback", "regular expression", "form", "date", "json", "closure"]
categories: "javascript"
---

### 获取URL中的信息

> [How to extract the hostname portion of a URL in JavaScript](http://stackoverflow.com/questions/1368264/how-to-extract-the-hostname-portion-of-a-url-in-javascript)

`http://localhost:8080/webtest1/index.html`
`window.location.host` -> `localhost:8080`
`window.location.hostname` -> `localhost`
`window.location.protocol` -> `http`
`window.location.port` -> `8080`
`window.location.origin` -> `http://localhost or http://localhost:8080`

----

### 格式化日期对象

> [Get String in YYYYMMDD format from JS date object?](http://stackoverflow.com/questions/3066586/get-string-in-yyyymmdd-format-from-js-date-object)

```js
Date.prototype.yyyymmdd = function() {
  var mm = this.getMonth() + 1;
  var dd = this.getDate();

  return [this.getFullYear(), !mm[1] && '0', mm, !dd[1] && '0', dd].join('');
};

var date = new Date();
date.yyyymmdd();
```

----

### js中检查对象是否为空

> [How do I test for an empty JavaScript object?](http://stackoverflow.com/questions/679915/how-do-i-test-for-an-empty-javascript-object)

ECMA5以后
```js
Object.keys(obj).length === 0 && JSON.stringify(obj) === JSON.stringify({});
```
ECMA5之前
```js
function isEmpty(obj) {
  for(var prop in obj) {
    if(obj.hasOwnProperty(prop))
      return false;
    }
  return true && JSON.stringify(obj) === JSON.stringify({});
}
```
jQuery:
```js
jQuery.isEmptyObject({});
```

### apply与call函数

> [js中apply方法的使用](http://www.cnblogs.com/delin/archive/2010/06/17/1759695.html)

apply方法能劫持另外一个对象的方法，继承另外一个对象的属性。`apply(obj, arr)`接收两个参数：
`obj`: 这个对象将代替Function类里this对象
`arr`: 将参数以数组的形式传递到Function中
```js
'use strict';

function Rectangle(width, height){
	this.width = width;
	this.height = height;
	this.area = function() {  return this.height * this.width; }
}

var rectangle = new Rectangle(6,4); 
console.log('rectangle area: ' + rectangle.area()); //rectangle area: 24

function Square(width){
	//Rectangle.call(this, width, width);
	Rectangle.call(this, [width, width]);
}

var square = new Square(4);
console.log('square area: ' + square.area());   //square area: 16
```

这样正方形获取了矩形的求面积方法，便可以计算出面积。
`apply`方法还可以用于其他环境，例如找出数组中最大的数字:
```js
console.log(Math.max.apply(null, [1,2,3,4,5]));
```
默认情况下`Math.max`方法只能接收两个参数，且不能为数组，如果使用上述写法，则可以求出数组中最大的数字来，同理也可以用作`Math.min`等函数。
`call(obj, arg1, arg2...)`方法与`apply(obj, arr)`方法类似，区别只是参数的不同。`call`后面跟的参数无限制，以需要获取的函数参数为准，而`apply`方法传入的是一个参数数组。

----

### 使用Javascript删除HTML元素

> [用Javascript删除HTML元素 ](http://blog.csdn.net/qingflyer/article/details/4025870)

```js
var div = document.getElementById("id");  
div.style.display = "none";         //隐藏而不删除  
div.parentNode.removeChild(div);    //删除
```

----

###  Javascript替换全部字符

> [JavaScript特殊字符替换及替换全部字符串](http://my.oschina.net/aicoding/blog/68960)
> [JQuery replace 替换全部](http://ys21426.blog.163.com/blog/static/11689204220127335137403/)

1.替换所有要替换字符
```js
var str = "$Hello World!$Hello World!$Hello World!";
//把所有的“Hello World!”替换为“Welcome you!”。“/g”是替换全部。
alert(str.replace(/Hello World!/g,"Welcome you!"));
```
2.替换所有要替换的特殊字符
```js
var str = "$Hello World!$Hello World!$Hello World!";
//利用正则表达式把所有的“$”替换为“#”。“$”为特殊字符，所以前面要加“\\”。
var regS = new RegExp("\\$","g");
alert(str.replace(regS,"#"));
```

----------

### JavaScript调整图片大小
> [用JavaScript调整图片大小](http://www.cnblogs.com/hun_dan/archive/2010/01/05/1639922.html)

jquery
```js
$(document).ready(function() {
    $('.post img').each(function() {
    var maxWidth = 100; // 图片最大宽度
    var maxHeight = 100;    // 图片最大高度
    var ratio = 0;  // 缩放比例
    var width = $(this).width();    // 图片实际宽度
    var height = $(this).height();  // 图片实际高度
 
    // 检查图片是否超宽
    if(width > maxWidth){
        ratio = maxWidth / width;   // 计算缩放比例
        $(this).css("width", maxWidth); // 设定实际显示宽度
        height = height * ratio;    // 计算等比例缩放后的高度 
        $(this).css("height", height * ratio);  // 设定等比例缩放后的高度
    }
 
    // 检查图片是否超高
    if(height > maxHeight){
        ratio = maxHeight / height; // 计算缩放比例
        $(this).css("height", maxHeight);   // 设定实际显示高度
        width = width * ratio;    // 计算等比例缩放后的高度
        $(this).css("width", width * ratio);    // 设定等比例缩放后的高度
    }
  });
});
```

javascript
```js
function ResizeImage(image, 插图最大宽度, 插图最大高度){
    if (image.className == "Thumbnail"){
        w = image.width;
        h = image.height;
 
        if( w == 0 || h == 0 ){
            image.width = maxwidth;
            image.height = maxheight;
        }else if (w > h){
            if (w > maxwidth) image.width = maxwidth;
        }else{
            if (h > maxheight) image.height = maxheight;
        }
        image.className = "ScaledThumbnail";
    }
}
```
需要手动为图片加上`class="Thumbnail"`

----------

### 函数多个参数
 
> [javascript返回多个参数](http://www.cnblogs.com/yzx99/archive/2008/08/05/1260561.html)
> [在JS方法中返回多个值的三种方法](http://www.cnblogs.com/oec2003/archive/2009/12/11/1621775.html)

```js
//数组方式
function f1(){
  var a = "hello";
  var b = "world";
  return [a,b];
}
//json方式
function f2(){
  return {"a":"hello","b":"world"};
}

function test(){
  alert(f1()[0]+" "+f1()[1]);
  console.log(f2()["a"]+" "+f2()["b"]);
}
```

----------

### url参数问题

> [Ajax传参之url中特殊字符的处理之血站八方](http://www.tuicool.com/articles/VJNbQb)
> [Parameters: Character decoding failed解决办法](http://blog.csdn.net/shikai0302/article/details/12205873)
> [url中参数带有双引号怎么办?](http://www.iteye.com/problems/15362)
> [url 参数中包含特殊字符](http://y-zjx.iteye.com/blog/1408085)
> [JavaScript encodeURIComponent() 函数](http://www.w3school.com.cn/jsref/jsref_encodeURIComponent.asp)

url包含了特殊字符需要转义，否则就会抛出`Parameters: Character decoding failed`错误。在提交前使用`replace`方法手动替换
```js
//这里将百分号进行转义，/g表示全部替换
var para = parameter.replace(/%/g,"%25");
```
需转义的部分特殊字符如下

| 特殊字符 | 代表意义 | 转码 |
| --- | --- | --- |
| + | +号表示空格 | %2B |
| 空格 | URL中的空格可以用+号 | %20 |
| / | 分隔目录和子目录 | %2F |
| ? | 分隔实际的 URL 和参数 | %3F |
| % | 指定特殊字符 | %25 |
| # | 表示书签 | %23 |
| & | URL 中指定的参数间的分隔符 | %26 |
| = | URL 中指定参数的值 | %3D |
| ! | 感叹号 | %21 |
| ( | 左括号 | %28 |
| ) | 右括号 | %29 |
| : | 冒号 | %3a |
| @ | 邮箱连接符 | %40 |
| . | 小数点 | %2e |
| , | 逗号 | %2c |


----------

### document.forms[0].submit()不提交表单
> [document.forms[0].submit()不提交表单](http://zochen.iteye.com/blog/836718)

如果确定能够触发`submit`方法的情况下那就是表单中有标签被命名为`submit`，改掉即可

----------

### 使用`getFullYear()`取代`getYear()`函数
> [JavaScript中getYear()显示错误问题](http://blog.csdn.net/makuiyu/article/details/7606934)

`getYear`、`getFullYear`、`getUTCFullYear`都是Javascript的Date对象的方法函数。其中`getYear()`方法出生较早，在早期也一直使用OK，可是在2000年后这个方法问题多多，因为在Firefox和Safari等浏览器上，getYear始终返回年份与1900 年之间的差，比如1998年返回98，而2009年则会显示109，如果大家都这么处理也好，要加一起加，微软自己在IE浏览器中把`getYear`给修正了，可Firefox（最新版本也没修正这个问题）还蒙在鼓里，仍老老实实的按照原有规则解析getYear，本来都可以指望用户自行修正，这样一来都没得用，于是`getFullYear`、`getUTCFullYear`就出生了。
1、`getYear()`函数
    使用`getYear()`方法可返回两位或四位数的年份，用`getYear()`返回的数并不一定是4位的！处于1900年和1999年间的`getYear()`方法返回的只有两位数。在此之前的或是在此之后的年份返回的都是四位数的，比如2009年，Javascript解析器应该是返回2009的，而浏览器则计算返回109。这应该是早期的约定，而IE埋头改掉了。该函数已经被逐渐废弃并不推荐使用。
```js
var d = new Date();
document.write(d.getYear());//IE输出2009，FIREFOX输出109
```
2、`getFullYea()`函数
    getFullYea`函数则不存在此问题。`getFullYear()`方法可返回一个四位数年份，这样大家（IE和FIREFOX等）都不需要运算，直接把解析值输出来即可。
```js
var d = new Date();
document.write(d.getFullYear());//IE输出2009，FIREFOX输出2009
```
3、getUTCFullYear()函数
    `getUTCFullYear()`函数则是根据UTC时间返回了四位数来代表年份。与`getFullYear()`方法理论角度是完全不同，虽然在大部分的时间里输出是相同的，但是假如当天日期是12月31日或1月1日，则`getUTCFullYear()`返回值与`getFullYear()`返回值就有可能不同，具体取决于当地时区和UTC通用时间之间的关系，也就是差值。
```js
var d = new Date();
document.write(d.getUTCFullYear());//IE输出2009，FIREFOX输出2009
```
比如在中国大陆、、香港、澳门、蒙古国、台湾、新加坡、马来西亚、菲律宾等地区的本地时间比UTC快8小时，记作UTC+8，意思就是比UTC时间快8小时。减的类似理解，比如UTC-10等。

----------

### js读取json数据
> [js读取json数据](http://www.cnblogs.com/qiantuwuliang/archive/2009/07/21/1527473.html)

方法一：函数构造定义法返回
```js
var strJSON = "{name:'json name'}";//得到的JSON
var obj = new Function("return" + strJSON)();//转换后的JSON对象
alert(obj.name);//json name
```
方法二：js中著名的eval函数
```js
var strJSON = "{name:'json name'}";//得到的JSON
var obj = eval( "(" + strJSON + ")" );//转换后的JSON对象
alert(obj.name);//json name
```
`eval`函数解析的表达式必须加一对括号括住

----------

### iframe中子父窗口互调的js方法

> [iframe中子父窗口互调的js方法](http://www.cnblogs.com/chinafine/archive/2011/09/15/2177746.html)

#### 父窗口调用iframe子窗口方法
HTML语法：`<iframe name="myFrame" src="child.html"></iframe>`
父窗口调用子窗口：`myFrame.window.functionName();`
子窗品调用父窗口：`parent.functionName();`
简单地说,也就是在子窗口中调用的变量或函数前加个`parent.`就行
#### 父窗口和子窗口相互的调用方法
父窗口调用子窗口：
```js
//IE
iframe_ID.iframe_document_object.object_attribute = attribute_value
//Firefox
window.frames["iframe_ID"].document.getElementById("iframe_document_object"­).object_attribute = attribute_value 
```
子窗口调用父窗口：
```js
//IE
parent.parent_document_object.object_attribute = attribute_value
//Firefox
parent.document.getElementById("parent_document_object").object_attribute = attribute_value 
```

----------

### 动态加载JS文件的三种方法
> [动态加载JS文件的三种方法](http://www.jb51.net/article/42942.htm)

1.重新生成一个script标签加载js文件
```js
function loadJs1(file){
    var head = $("head").remove("script[role='reload']");
    $("<script></script>").attr({ role: 'reload', src: file, type: 'text/javascript' }).appendTo(head);
}
```
2.需要给script标签定义一个id
```js
function loadJs2(id,newJS){
    var oldjs = null;
    var t = null;
    var oldjs = document.getElementById(id);
    if(oldjs) oldjs.parentNode.removeChild(oldjs);
    var scriptObj = document.createElement("script");
    scriptObj.src = newJS;
    scriptObj.type = "text/javascript";
    scriptObj.id = id;
    document.getElementsByTagName("head")[0].appendChild(scriptObj);
}
```
3.使用jquery的`getScript`方法
```js
$.getScript('test.js',function(){
   //可以在这里运行载入的js文件里面的函数
});
```

----------

### javascript回调函数

> [JavaScript回调(callback)函数概念自我理解及示例](http://www.jb51.net/article/39497.htm)

```js
function f1(callback){
  console.log("f1");
  callback();
}

function c1(){
  console.log("c1");
}

function c2(){
  console.log("c2");
}

function test(){
  f1(c1());
  f1(c2());
}
```

在以上例子中，`f1()`中接入的参数为函数时则为回调函数。当f1函数执行完毕后可以执行传入的函数。

----------

### javascript 闭包

> [javascript深入理解js闭包](http://www.jb51.net/article/24101.htm)

#### 一、变量的作用域
要理解闭包，首先必须理解Javascript特殊的变量作用域。
变量的作用域无非就是两种：**全局变量**和**局部变量**。
Javascript语言的特殊之处，就在于函数内部可以直接读取全局变量。
```js
var n=999;
function f1(){
　alert(n);
}
f1(); // 999
```
另一方面，在函数外部自然无法读取函数内的局部变量。
```js
function f1(){
　var n=999;
}
alert(n); // error
```
这里有一个地方需要注意，函数内部声明变量的时候，一定要使用var命令。如果不用的话，你实际上声明了一个全局变量！ 
```js
function f1(){
　n=999;
}
f1();
alert(n); // 999
```

----------

#### 二、如何从外部读取局部变量？
出于种种原因，我们有时候需要得到函数内的局部变量。但是，前面已经说过了，正常情况下，这是办不到的，只有通过变通方法才能实现。
那就是在函数的内部，再定义一个函数。 
```js
function f1(){
　n=999;
　function f2(){
　　alert(n); // 999
　}
}
```
在上面的代码中，函数f2就被包括在函数f1内部，这时f1内部的所有局部变量，对f2都是可见的。但是反过来就不行，f2内部的局部变量，对f1 就是不可见的。这就是Javascript语言特有的“**链式作用域**”结构（chain scope）: 子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。

既然f2可以读取f1中的局部变量，那么只要把f2作为返回值，我们不就可以在f1外部读取它的内部变量了吗！ 
```js
function f1(){
　n=999;
　function f2(){
　　alert(n);
　}
  return f2;
}
var result=f1();
result(); // 999
```

----------

#### 三、闭包的概念
上一节代码中的f2函数，就是闭包。

各种专业文献上的“闭包”（closure）定义非常抽象，很难看懂。我的理解是，闭包就是能够读取其他函数内部变量的函数。
由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成“定义在一个函数内部的函数”。
所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁

**官方**的解释是：闭包是一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。

当内部函数 在定义它的作用域 的外部 被引用时,就创建了该内部函数的闭包 ,如果内部函数引用了位于外部函数的变量,当外部函数调用完毕后,这些变量在内存不会被 释放,因为闭包需要它们. 

----------

#### 四、闭包的用途
闭包可以用在许多地方。它的最大用处有两个，**一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中**。
```js
function f1(){
　var n=999;
　nAdd=function(){n+=1}
　　function f2(){
　　　alert(n);
　　}
　　return f2;
　}
　var result=f1();
　result(); // 999
　nAdd();
　result(); // 1000
```
在这段代码中，result实际上就是闭包f2函数。它一共运行了两次，第一次的值是999，第二次的值是1000。这证明了，函数f1中的局部变量n一直保存在内存中，并没有在f1调用后被自动清除。
为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。
这段代码中另一个值得注意的地方，就是`nAdd=function(){n+=1}`这一行，首先在nAdd前面没有使用var关键字，因此 nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。

----------

#### 五、使用闭包的注意点
1）由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
2）闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。 


----------

### javascript事件
#### `input`事件
> 来自 
> [JavaScript处理input的keyup事件](http://blog.sina.com.cn/s/blog_6aaf309f01013o4t.html)
> [监听输入框值的即时变化onpropertychange、oninput](http://bbs.blueidea.com/thread-2966314-1-1.html)

完整的 key press 过程分为两个部分，按键被按下，然后按键被松开并复位。当按钮被松开时，发生 keyup 事件。它发生在当前获得焦点的元素上。`keyup()` 方法触发 keyup 事件，或规定当发生 keyup 事件时运行的函数。

 IE下，当一个HTML元素的属性改变的时候，都能通过**onpropertychange**来即时捕获。
onchange在属性值改变时还必须使得当前元素失去焦点(onblur)才可以激活该事件。在其他浏览器下可以使用**oninput**事件来达到同样的效果。


----------

#### `onbeforeunload`事件
> 来自 
> [离开页面前onbeforeunload事件在火狐的兼容并且提交不触发](http://blog.csdn.net/shy_snow/article/details/5408327)
> [onbeforeunload 在Firefox中的兼容问题](http://cssha.com/onbeforeunload-firefox/)
> [结合jQuery的unload方法实现JS退出页面弹出对话框](http://www.rainweb.cn/article/113.html)

`onbeforeunload`会在网页即将关闭，返回，刷新等条件下触发,可以用于弹出对话框。firefox拥有自己的弹出框，因此需要特殊处理
```js
window.onbeforeunload=function checkLeave(e){
  //firefox特殊处理
  var evt = e ? e : (window.event ? window.event : null);
  evt.returnValue='离开会使编写的内容丢失。';
}
```

----------

### javascript正则
#### 只能输入数字及两位小数点

> [正则表达式实现只能输入.和数字并且只能有两位小数](http://tianshi-min.blog.163.com/blog/static/35139270201302251859933/?suggestedreading)
> [js 只能输入数字和小数点的文本框改进版](http://www.jb51.net/article/17782.htm)

```js
/*判断输入数字*/
function clearNoNum(event, obj) {
	// 响应鼠标事件，允许左右方向键移动
	event = window.event || event;
	if (event.keyCode == 37 | event.keyCode == 39) { return; }
	// 先把非数字的都替换掉，除了数字和.
	obj.value = obj.value.replace(/[^\d.]/g, "");
	// 必须保证第一个为数字而不是.
	obj.value = obj.value.replace(/^\./g, "");
	// 保证只有出现一个.而没有多个.
	obj.value = obj.value.replace(/\.{2,}/g, ".");
	// 保证.只出现一次，而不能出现两次以上
	obj.value = obj.value.replace(".", "$#$").replace(/\./g, "").replace("$#$",".");
}

function checkNum(obj){ 
	// 为了去除最后一个.
	obj.value = obj.value.replace(/\.$/g,""); 
}
```
```html
<input type="text" onkeyup="clearNoNum(event,this)" onblur="checkNum(this)">
```


----------

#### 验证身份证号

> [身份证号,出生日期等的js正则表达式验证](http://snailwong.iteye.com/blog/400298)

简单的正则表达式：
```
reg_match("/^(\d{18,18}|\d{15,15}|\d{17,17}x)$/",$id_card) 
preg_match("/^(\d{6})(18|19|20)?(\d{2})([01]\d)([0123]\d)(\d{3})(\d|X)?$/",$id_card)
reg_match("/(^\d{15}$/)|(\d{17}(?:\d|x|X)$/),$id_card)
```
严格的验证
```js
function validateIdCode(num) {
    num = num.toUpperCase();
	//身份证号码为15位或者18位，15位时全为数字，18位前17位为数字，最后一位是校验位，可能为数字或字符X。  
	if (!(/(^\d{15}$)|(^\d{17}([0-9]|X)$)/.test(num))) {
		alert('输入的身份证号长度不对，或者号码不符合规定！\n15位号码应全为数字，18位号码末位可以为数字或X。');
		return false;
	}
	//校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
	//下面分别分析出生日期和校验位
	var len, re;
	len = num.length;
	if (len == 15) {
		re = new RegExp(/^(\d{6})(\d{2})(\d{2})(\d{2})(\d{3})$/);
		var arrSplit = num.match(re);

		//检查生日日期是否正确
		var dtmBirth = new Date('19' + arrSplit[2] + '/' + arrSplit[3] + '/'
				+ arrSplit[4]);
		var bGoodDay;
		bGoodDay = (dtmBirth.getYear() == Number(arrSplit[2]))
				&& ((dtmBirth.getMonth() + 1) == Number(arrSplit[3]))
				&& (dtmBirth.getDate() == Number(arrSplit[4]));
		if (!bGoodDay) {
			alert('输入的身份证号里出生日期不对！');
			return false;
		} else {
			//将15位身份证转成18位
			//校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
			var arrInt = new Array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5,
					8, 4, 2);
			var arrCh = new Array('1', '0', 'X', '9', '8', '7', '6', '5', '4',
					'3', '2');
			var nTemp = 0, i;
			num = num.substr(0, 6) + '19' + num.substr(6, num.length - 6);
			for (i = 0; i < 17; i++) {
				nTemp += num.substr(i, 1) * arrInt[i];
			}
			num += arrCh[nTemp % 11];
			return num;
		}
	}
	if (len == 18) {
		re = new RegExp(/^(\d{6})(\d{4})(\d{2})(\d{2})(\d{3})([0-9]|X)$/);
		var arrSplit = num.match(re);

		//检查生日日期是否正确
		var dtmBirth = new Date(arrSplit[2] + "/" + arrSplit[3] + "/"
				+ arrSplit[4]);
		var bGoodDay;
		bGoodDay = (dtmBirth.getFullYear() == Number(arrSplit[2]))
				&& ((dtmBirth.getMonth() + 1) == Number(arrSplit[3]))
				&& (dtmBirth.getDate() == Number(arrSplit[4]));
		if (!bGoodDay) {
			//alert(dtmBirth.getYear());
			//alert(arrSplit[2]);
			alert('输入的身份证号里出生日期不对！');
			return false;
		} else {
			//检验18位身份证的校验码是否正确。
			//校验位按照ISO 7064:1983.MOD 11-2的规定生成，X可以认为是数字10。
			var valnum;
			var arrInt = new Array(7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5,
					8, 4, 2);
			var arrCh = new Array('1', '0', 'X', '9', '8', '7', '6', '5', '4',
					'3', '2');
			var nTemp = 0, i;
			for (i = 0; i < 17; i++) {
				nTemp += num.substr(i, 1) * arrInt[i];
			}
			valnum = arrCh[nTemp % 11];
			if (valnum != num.substr(17, 1)) {
				alert('18位身份证的校验码不正确!');
				return false;
			}
			return num;
		}
	}
	return false;
}
```

----------

#### 正则表达式大全
> [正则表达式 ———— 大全](http://blog.csdn.net/chaoa888/article/details/7411840)

正则表达式及限制字数
```js
"^\d+$"　　//非负整数（正整数 + 0）  
^(?:0|[1-9]\d{0,2})(\.\d)?$(判断数字小于1000,小数位数只能有1位,不是负数的正则表达式)
"^[0-9]*[1-9][0-9]*$"　　//正整数 
"^((-\d+)|(0+))$"　　//非正整数（负整数 + 0） 
"^-[0-9]*[1-9][0-9]*$"　　//负整数 
"^-?\d+$"　　　　//整数 
"^\d+(\.\d+)?$"　　//非负浮点数（正浮点数 + 0） 
"^(([0-9]+\.[0-9]*[1-9][0-9]*)|([0-9]*[1-9][0-9]*\.[0-9]+)|([0-9]*[1-9][0-9]*))$"　　//正浮点数 
"^((-\d+(\.\d+)?)|(0+(\.0+)?))$"　　//非正浮点数（负浮点数 + 0） 
"^(-(([0-9]+\.[0-9]*[1-9][0-9]*)|([0-9]*[1-9][0-9]*\.[0-9]+)|([0-9]*[1-9][0-9]*)))$"　　//负浮点数 
"^(-?\d+)(\.\d+)?$"　　//浮点数 
"^[A-Za-z]+$"　　//由26个英文字母组成的字符串 
"^[A-Z]+$"　　//由26个英文字母的大写组成的字符串 
"^[a-z]+$"　　//由26个英文字母的小写组成的字符串 
"^[A-Za-z0-9]+$"　　//由数字和26个英文字母组成的字符串 
"^\w+$"　　//由数字、26个英文字母或者下划线组成的字符串 
"^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$"　　　　//email地址 
"^[a-zA-z]+://(\w+(-\w+)*)(\.(\w+(-\w+)*))*(\?\S*)?$"　　//url
/^(d{2}|d{4})-((0([1-9]{1}))|(1[1|2]))-(([0-2]([1-9]{1}))|(3[0|1]))$/   // 年-月-日
/^((0([1-9]{1}))|(1[1|2]))/(([0-2]([1-9]{1}))|(3[0|1]))/(d{2}|d{4})$/   // 月/日/年
"^([w-.]+)@(([[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.)|(([w-]+.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(]?)$"   //Emil
"(d+-)?(d{4}-?d{7}|d{3}-?d{8}|^d{7,8})(-d+)?"     //电话号码
"^(d{1,2}|1dd|2[0-4]d|25[0-5]).(d{1,2}|1dd|2[0-4]d|25[0-5]).(d{1,2}|1dd|2[0-4]d|25[0-5]).(d{1,2}|1dd|2[0-4]d|25[0-5])$"   //IP地址
```

匹配中文字符的正则表达式： `[\u4e00-\u9fa5]`
匹配双字节字符(包括汉字在内)：`[^\x00-\xff]`
匹配空行的正则表达式：`\n[\s| ]*\r`
匹配HTML标记的正则表达式：`/<(.*)>.*<\/\1>|<(.*) \/>/`
匹配首尾空格的正则表达式：`(^\s*)|(\s*$)`
匹配Email地址的正则表达式：`\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*`
匹配网址URL的正则表达式：`^[a-zA-z]+://(\\w+(-\\w+)*)(\\.(\\w+(-\\w+)*))*(\\?\\S*)?$`
匹配帐号是否合法(字母开头，允许5-16字节，允许字母数字下划线)：`^[a-zA-Z][a-zA-Z0-9_]{4,15}$`
匹配国内电话号码：`(\d{3}-|\d{4}-)?(\d{8}|\d{7})?`
匹配腾讯QQ号：`^[1-9]*[1-9][0-9]*$`

```js
//用正则表达式限制只能输入中文
onkeyup="value=value.replace(/[^u4E00-u9FA5]/g,'')" onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^u4E00-u9FA5]/g,''))"


//用正则表达式限制只能输入全角字符
onkeyup="value=value.replace(/[^uFF00-uFFFF]/g,'')" onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^uFF00-uFFFF]/g,''))"


//用正则表达式限制只能输入数字
onkeyup="value=value.replace(/[^d]/g,'') "onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^d]/g,''))"


//用正则表达式限制只能输入数字和英文
onkeyup="value=value.replace(/[W]/g,'') "onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^d]/g,''))"
```

元字符及其在正则表达式上下文中的行为：
`\` 将下一个字符标记为一个特殊字符、或一个原义字符、或一个后向引用、或一个八进制转义符。
`^` 匹配输入字符串的开始位置。如果设置了 RegExp 对象的Multiline 属性，`^` 也匹配 `\n` 或 `\r` 之后的位置。
`$` 匹配输入字符串的结束位置。如果设置了 RegExp 对象的Multiline 属性，`$` 也匹配 `\n` 或 `\r` 之前的位置。
`*` 匹配前面的子表达式零次或多次。
`+` 匹配前面的子表达式一次或多次。`+` 等价于 `{1,}`。
`?` 匹配前面的子表达式零次或一次。`?` 等价于 `{0,1}`。
`{n}` n 是一个非负整数，匹配确定的n 次。
`{n,}` n 是一个非负整数，至少匹配n 次。
`{n,m}` m 和 n 均为非负整数，其中*n <= m*。最少匹配 n 次且最多匹配 m 次。在逗号和两个数之间不能有空格。
`?` 当该字符紧跟在任何一个其他限制符 (*, +, ?, {n}, {n,}, {n,m}) 后面时，匹配模式是非贪婪的。非贪婪模式尽可能少的匹配所搜索的字符串，而默认的贪婪模式则尽可能多的匹配所搜索的字符串。
`.` 匹配除 `\n` 之外的任何单个字符。要匹配包括 `\n` 在内的任何字符，请使用象 `[.\n]` 的模式.(pattern) 匹配pattern 并获取这一匹配。
`(?:pattern)` 匹配pattern 但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用
`(?=pattern)` 正向预查，在任何匹配 pattern 的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。
`(?!pattern)` 负向预查，与`(?=pattern)`作用相反
`x|y` 匹配 x 或 y
`[xyz]` 字符集合
`[^xyz]` 负值字符集合
`[a-z]` 字符范围，匹配指定范围内的任意字符
`[^a-z]` 负值字符范围，匹配任何不在指定范围内的任意字符
`\b` 匹配一个单词边界，也就是指单词和空格间的位置
`\B` 匹配非单词边界
`\cx` 匹配由x指明的控制字符
`\d` 匹配一个数字字符。等价于 `[0-9]`
`\D` 匹配一个非数字字符。等价于 `[^0-9]`
`\f` 匹配一个换页符。等价于 `\x0c` 和 `\cL`
`\n` 匹配一个换行符。等价于 `\x0a` 和 `\cJ`
`\r` 匹配一个回车符。等价于 `\x0d` 和 `\cM`
`\s` 匹配任何空白字符，包括空格、制表符、换页符等等。等价于`[ \f\n\r\t\v]`
`\S` 匹配任何非空白字符。等价于 `[^ \f\n\r\t\v]`
`\t` 匹配一个制表符。等价于 `\x09` 和 `\cI`
`\v` 匹配一个垂直制表符。等价于 `\x0b` 和 `\cK`
`\w` 匹配包括下划线的任何单词字符。等价于`[A-Za-z0-9_]`
`\W` 匹配任何非单词字符。等价于 `[^A-Za-z0-9_]`
`\xn` 匹配 n，其中 n 为十六进制转义值。十六进制转义值必须为确定的两个数字长
`\num` 匹配 num，其中num是一个正整数。对所获取的匹配的引用
`\n` 标识一个八进制转义值或一个后向引用。如果 `\n` 之前至少 n 个获取的子表达式，则 n 为后向引用。否则，如果 n 为八进制数字 (0-7)，则 n 为一个八进制转义值
`\nm` 标识一个八进制转义值或一个后向引用。如果 `\nm` 之前至少有nm 个获取得子表达式，则 nm 为后向引用。如果 `\nm` 之前至少有 n 个获取，则 n 为一个后跟文字 m 的后向引用。如果前面的条件都不满足，若 n 和 m 均为八进制数字 (0-7)，则 `\nm` 将匹配八进制转义值 nm
`\nml` 如果 n 为八进制数字 (0-3)，且 m 和 l 均为八进制数字 (0-7)，则匹配八进制转义值 nml
`\un` 匹配 n，其中 n 是一个用四个十六进制数字表示的Unicode字符