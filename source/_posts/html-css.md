title: "html标签与css样式表"
date: 2015-05-10 00:19:58
tags: "css"
categories: ["web","css"]
---

###  html实线边框的表格样式定义 

> [html实线边框的表格样式定义](http://blog.csdn.net/henrytsu/article/details/3446384)

```css
table
{
  border-collapse:collapse;
  border-left:solid 1 #000000; border-top:solid 1 #000000;
  padding:5px;
}


th
{
  border-right:solid 1 #000000;
  border-bottom:solid 1 #000000;
  background-color: Silver;
}

td
{
  font:normal;
  border-right:solid 1 #000000;
  border-bottom:solid 1 #000000;
  background-color: transparent;
}
```

----------

### 表格奇偶行不同颜色
> 来自 [纯CSS table 表格奇偶行不同颜色实现](http://www.cnblogs.com/sanmen/archive/2012/08/09/2631011.html)

```css
.table-striped tbody tr:nth-child(odd) td {
  background-color: Red;
}
```
```html
<table class="table-striped">
</table>
```

----------

### CSS发光边框文本框效果
> 来自 [CSS发光边框文本框效果](http://blog.netsh.org/posts/css-input-border-light-box-effect_533.netsh.html)

```css
input[type=text]:focus,input[type=password]:focus,textarea:focus{
  transition:border linear .2s,box-shadow linear .5s;
  -moz-transition:border linear .2s,-moz-box-shadow linear .5s;
  -webkit-transition:border linear .2s,-webkit-box-shadow linear .5s;
  outline:none;border-color:rgba(241,39,242,.75);
  box-shadow:0 0 8px rgba(241,39,232,.5);
  -moz-box-shadow:0 0 8px rgba(241,39,232,.5);
  -webkit-box-shadow:0 0 8px rgba(241,39,232,3);
}
```
其中的RGB色彩可以根据个人口味进行改变

----

### html table td边框效果

> [html table td边框效果](http://hi.baidu.com/9prior/item/60b47eb61cd3b8941846970e)

同时用样式表为 table、td 指定了边框后，可能会发生重叠，这取决于 `border-collapse`
```html
<table style="border:1px solid red;border-collapse:collapse;">
  <tr>
    <td style="border:1px solid blue;">&nbsp;</td>
    <td style="border:1px solid blue;">&nbsp;</td>
    <td style="border:1px solid blue;">&nbsp;</td>
    <td style="border:1px solid blue;">&nbsp;</td>
  </tr>
</table>
```
在发生重叠时，Firefox 是用 td 覆盖 table 的，而 IE 是用 table 覆盖 td 的
```html
<table border="1" bordercolor="#FF9966" >
  <tr>
    <td width="102" style="border-right-style:none">隐藏右边框</td>
    <td width="119" style="border-left-style:none">隐藏左边框</td>
  </tr>
  <tr>
    <td style="border-top-style:none">隐藏上边框</td>
    <td style="border-bottom-style:none">隐藏下边框</td>
  </tr>
</table>

<table>
  <tr>
    <td style="border-right:#cccccc solid 1px;">显示右边框</td>
    <td style="border-left:#cccccc solid 1px;">显示左边框</td>
    <td style="border-top:#cccccc solid 1px;">显示上边框</td>
    <td style="border-bottom:#cccccc solid 1px;">显示下边框</td>
  </tr>
</table>

<table>
  <tr>
    <td style="border-right : thin dashed blue;">右边框显示细虚线</td>
    <td style="border-bottom: thick dashed yellow;">左边框显示粗虚线</td>
    <td style="border-top: double green;">上边框显示两条线</td>
    <td style="border-left: dotted red;">下边框显示点</td>
  </tr>
</table>
```
