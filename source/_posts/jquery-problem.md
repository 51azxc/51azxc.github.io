title: "jquery填坑"
date: 2015-05-11 17:51:19
tags: ["upload", "json"]
categories: ["javascript", "jQuery"]
---

### head.insertBefore( script, head.firstChild ); 

> [多次Jquery引发head.insertBefore( script, head.firstChild ); ](http://blog.csdn.net/diligentcatrich/article/details/5903554)

多次载入jquery是指在一个页面里引用jquery，而在DIV中载入的那个页面中就不要再引用jquery了。如果引用则会触发head.insertBefore( script, head.firstChild );的错误。

----------

### TypeError: invalid 'in' operand obj

> [有关TypeError: invalid 'in' operand obj的错误](http://blog.csdn.net/lwx2615/article/details/9668777)

使用each解析json时候，如果会报错TypeError: invalid 'in' operand obj则需要调用`$.parseJSON`方法
```js
$.ajax({
  type: "POST",
  url: url,
  dataype: "json",
  success: function (data) {
    obj= $.parseJSON(data);
    $.each( obj, function(key, val) {
      
    });
  }
});
```
官网对parseJSON的描述是：*Takes a well-formed JSON string and returns the resulting JavaScript object.*

----------

### 解决JQuery.trim()函数ie下报错的问题

> [解决JQuery.trim()函数ie下报错的问题](http://vsp.iteye.com/blog/1262441)

```js
//这种写法在firefox下有效，但是在ie下无效
console.log($("input").val().trim()!="");
//ie的写法
alert($.trim($("input").val())!="");
```

----------

### x-editable更改默认的空白字段的填充文字

> [x-editable (twitter bootstrap): how to change the empty value?](http://stackoverflow.com/questions/19494605/x-editable-twitter-bootstrap-how-to-change-the-empty-value)

使用属性`emptytext`修改
```js
emptytext: 'space'
```

----

### jQuery.handleError is not a function

> [jquery.form 无刷新上传文件报错(jQuery.handleError is not a function)，gb2312下中文乱码问题](http://hi.baidu.com/wangsen911/item/5ddf775744c44d01e7c4a51a)

jQuery.handleError is not a function 原因是：

1.handlerError只在jquery-1.4.2之前的版本中存在，jquery-1.6 和1.7中都没有这个函数了。

2.如果返回的dataType: "json", 是json格式的，则还需要添加httpData方法。

因此在jquery高级版本中将这个函数添加上 ，问题解决。

需要添加的代码：
```js
; (function ($) {
jQuery.extend({
    handleError: function (s, xhr, status, e) {
        if (s.error) {
            s.error.call(s.context || s, xhr, status, e);
        }
        if (s.global) {
            (s.context ? jQuery(s.context) : jQuery.event).trigger("ajaxError", [xhr, s, e]);
        }
    },
    httpData: function (xhr, type, s) {
        var ct = xhr.getResponseHeader("content-type"),
xml = type == "xml" || !type && ct && ct.indexOf("xml") >= 0,
data = xml ? xhr.responseXML : xhr.responseText;
        if (xml && data.documentElement.tagName == "parsererror")
            throw "parsererror";
        if (s && s.dataFilter)
            data = s.dataFilter(data, type);
        if (typeof data === "string") {
            if (type == "script")
                jQuery.globalEval(data);
            if (type == "json")
                data = window["eval"]("(" + data + ")");
        }
        return data;
    }
});
```

中文乱码问题则是 在gb2312字符编码格式下 需要修改js源码：
```js
$.fn.param=function( a ) {

   var encode=function(v){//如果包含中文就escape,避免重复escape)
     return /[^\x00-\xff]/g.test(v)?escape(v):v;
    }
   var s = [];
   // If an array was passed in, assume that it is an array
   // of form elements
   if ( a.constructor == Array || a.jquery )
    // Serialize the form elements
    jQuery.each( a, function(){
     s.push( encode(this.name) + "=" + encode( this.value ) );
    });

   // Otherwise, assume that it's an object of key/value pairs
   else
    // Serialize the key/values
    for ( var j in a )
     // If the value is an array then the key names need to be repeated
     if ( a[j] && a[j].constructor == Array )
      jQuery.each( a[j], function(){
       s.push( encode(j) + "=" + encode( this ) );
      });
     else
      s.push( encode(j) + "=" + encode( a[j] ) );

   // Return the resulting serialization
   return s.join("&").replace(/%20/g, "+");
};
```

----

### Spring Jackson AjaxFileUpload 没有执行回调函数的解决办法

> [Spring Jackson AjaxFileUpload 没有执行回调函数的解决办法](http://www.iteye.com/topic/1118960)

修改ajaxfileupload的源码
```js
if ( type == "json" )
  eval( "data = " + data );
```
这一句修改为
```js
if ( type == "json" )
  data=eval("("+data.replace("<pre>","").replace("</pre>","")+")");
```

----------

### ajaxFileUpload plugin上传文件 chrome、Firefox中出现`SyntaxError:unexpected token <`

> [ajaxFileUpload plugin上传文件 chrome、Firefox中出现`SyntaxError:unexpected token <`](http://liwx2000.iteye.com/blog/1540321)

因为Server端的Response上加上了`contentType="application/json"`。但有时后端这么做是必须的，所以修改ajaxFileUpload源码，将`<pre></pre>`标签去掉
```js
uploadHttpData: function( r, type ) {  
        var data = !type;  
        data = type == "xml" || data ? r.responseXML : r.responseText;  
        // If the type is "script", eval it in global context  
        if ( type == "script" )  
            jQuery.globalEval( data );  
        // Get the JavaScript object, if JSON is used.  
        if ( type == "json" ) {  
             ////////////以下为新增代码///////////////  
             data = r.responseText;  
             var start = data.indexOf(">");  
             if(start != -1) {  
               var end = data.indexOf("<", start + 1);  
               if(end != -1) {  
                 data = data.substring(start + 1, end);  
                }  
             }  
              ///////////以上为新增代码///////////////  
              eval( "data = " + data);  
        }  
        // evaluate scripts within html  
        if ( type == "html" )  
            jQuery("<div>").html(data).evalScripts();  
  
        return data;  
    }
```
