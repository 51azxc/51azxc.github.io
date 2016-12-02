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
