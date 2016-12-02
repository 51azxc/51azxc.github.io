title: "html table td边框效果"
date: 2015-04-21 23:27:10
tags: "table"
categories: ["web", "html"]
---

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
