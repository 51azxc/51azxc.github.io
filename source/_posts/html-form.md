title: "html表单"
date: 2015-05-09 23:49:09
tags: "form"
categories: ["web", "html"]
---

### 禁止文本框的记忆性输入
> [禁止文本框的记忆性输入 ,input ,textbox](http://www.cnblogs.com/ajax2008/archive/2011/07/31/2122701.html)

设置表单属性`AUTOCOMPLETE`为**OFF**
```html
<form method="post" AUTOCOMPLETE="OFF">
```
设置单个输入框
```html
<input type="text" AUTOCOMPLETE="OFF">
```
如果要禁止文本框使用输入法，可以把在它的样式中添加 ime-mode : disabled 即可，但是这样并不能禁止输入汉字，因为用户还是可以通过复制粘贴输入汉字的
```html
<input type="text" style="ime-mode: disabled;">
```

----------

### 提交表单后清空输入框内容

> [如何使表单提交后,清空表单中文本框的内容](http://bbs.51js.com/thread-5147-1-1.html)

提交按钮的`onClick`方法指定表单提交之后执行`reset`方法
```html
<form id="form1" method="post">
  <input type="text" id="text1">
  <input type="button" value="Submit" onClick="form1.submit();form1.reset();">
</form>
```

----------

### 让select下拉列表只读

> [如何让select下拉选择只读](http://bbs.csdn.net/topics/20196208)
> [如何把select选项给只读，让他不可选，但数据还是保存在下拉表中](http://bbs.csdn.net/topics/20418138)

```html
<select onchange="selectedIndex=this.defaultChecked">
  <option>1</option>
  <option>2</option>
  <option>3</option>
</select>

<select onfocus="this.blur()" onmouseover="this.setCapture()" onmouseout="this.releaseCapture()"> 
```

----------

### 强制页面图片刷新

> [不刷新页面，如何强制图片刷新？](http://bbs.csdn.net/topics/70499996)

只要保证每次src的字符串不同就会重取
```js
var date=new Date();
img1.src=图片地址+"?"+date.toLocaleString();
//or
img1.src=img1.src + "?" + (new Date().getTime()) 
```

----------

### 用get方法丢失参数

> [用get方法丢失参数啦 请指教](http://bbs.csdn.net/topics/300046767)

如果提交的表单`action`中跟的参数与表单中的某个参数重名，则会造成参数丢失的情况，这时设置表单的`method`方法为`post`即可。或者写入隐藏域中，即使用`<input type="hidden">`

----

### 表单中的button类型问题

> [button会自动提交表单吗](http://bbs.csdn.net/topics/390982400)

如果在表单中使用了`button`标签，一定要为其指定`type`类型，如果不指定，IE默认其为`button`类型，而FF,Chrome等浏览器则认定其默认类型为`Submit`,因此点击`button`会提交表单。为了预防表单提交，可以设置表单的`onSubmit`方法返回`false`，然后再指定`button`的点击事件为提交方法：
```html
<form action="" method="post" id="form1" onSubmit="return false">
  <button type="button" onClick="submitForm()">
</form>
```
