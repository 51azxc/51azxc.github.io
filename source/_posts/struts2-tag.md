title: "Struts2 部分标签"
date: 2015-05-09 01:07:11
tags: ["collection", "form"]
categories: ["java", "struts2"]
---

### Struts2 部分标签
#### `iterator`标签

> [struts2中iterator标签的相关使用](http://blog.csdn.net/oxcow/article/details/4516283)
> [struts2 的标签遍历：`list<map<String,String>>`](http://www.iteye.com/problems/23667)

`iterator`标签可以用来遍历数组，最简单的使用
```
List<String> days = ["Mon","Tue","Wed","Thu","Fri"];

<s:iterator value="days">
  <s:property />
</s:iterator>
```

<!-- more -->

在`iterator`中，`top`可以表示当前元素，也可以携程`[0].top`.[0]代表整个栈对象
```xml
<s:iterator value="days">
  <s:if test="top!='Mon'">
    <s:property />
  </s:if>
</s:iterator>
```
`iterator`的`status`属性有如下方法:
* `count`: int - 返回当前迭代位置的计数(从1开始)
* `index`: int - 返回当前迭代位置的编号(从0开始)
* `first`: boolean - 如果当前迭代位置是第一位时返回true
* `last`: boolean - 如果当前迭代位置是最后一位时返回true
* `even`: boolean - 如果当前迭代位置是偶数返回true
* `odd`: boolean - 如果当前迭代位置是奇数返回true
* `modulus(operand : int)`: int - 返回当前计数(从1开始)与指定操作数的模数

```xml
<s:iterator value="days" status="day">
  <s:if test="#day.first">
    <s:property />
  </s:if>
</s:iterator>
```

最后再来看下在iterator中调用value stack的用法。
假定countries是一个List对象，每一个country有一个name属性和一个citys List对象，并且每一个city也有一个name属性。那么我们想要在迭代citys时访问所属country的name属性就的用如下方式：
```xml
<s:iterator value="countries">  
    <s:iterator value="cities">  
        <s:property value="name"/>, <s:property value="[1].name"/><br>  
    </s:iterator>  
</s:iterator>
```
* 这里的 `<s:property value="name"/>`取的是ctiy.name;`<s:property value="[1].name"/>`取得是country.name
* `<s:property value="[1].name"/>` 等价于 `<s:property value="[1].top.name"/>`
* city处于当前栈，即top或者[0],而[1]指明了外层iterator对象，即country
* '[n]'标记引用开始位置为n的子栈（sub-stack），而不仅仅是位置n处的对象。因此'[0]'代表整个栈，而'[1]'是除top对象外所有的栈元素。

`iterator`遍历：list<map<String,String>>
```xml
<s:iterator id="map" value="userList" status="user_state">  
    <s:iterator value="userList[#user_state.index]">   
        Key : <s:property value="key" />  
        Value : <s:property value="value" />  
        <br>  
    </s:iterator>  
</s:iterator>
```

----------

#### `select`标签
> [struts2中s:select标签的使用](http://blog.csdn.net/moliqin/article/details/3753570)

`headerKey headerValue` 为设置缺省值
`listKey`为option的value,`listValue`为option显示的选项

普通数组
```
<s:select list="{'aa','bb','cc'}"
          theme="simple" 
          headerKey="00" 
          headerValue="00">
</s:select>
```

key-value映射
```
<s:select list="#{1:'aa',2:'bb',3:'cc'}"  
          label="abc" 
          listKey="key" 
          listValue="value"  
          headerKey="0" 
          headerValue="aabb">
</s:select>
```

```jsp
<%
java.util.HashMap map = new java.util.LinkedHashMap();
map.put(1,"aaa");
map.put(2,"bbb");
map.put(3,"ccc");
request.setAttribute("map",map);
request.setAttribute("aa","2");
%>
<s:select list="#request.map"  
          label="abc" 
          listKey="key" 
          listValue="value" 
          value="#request.aa"  
          headerKey="0" 
          headerValue="aabb">
</s:select>
```

----------

#### Struts2 s:checkboxlist 国际化 
> [Struts2 s:checkboxlist 国际化](http://blog.csdn.net/crazy_kis/article/details/4765937)

使用`getText(key)`方法即可,key为国际化资源文件中的key值
```
<s:checkboxlist name="adminUserRoles" 
                theme="simple" 
                list="#{1:getText('aa'),2:getText('bb'),3:getText('cc')}"  />
```


----------

#### `checkbox`标签

> [Struts2标签之Checkbox详解](http://www.blogjava.net/SpartaYew/archive/2011/05/19/350594.html)

`<s:checkbox>`有如下属性:
`id`: 标签id
`name`: 标签name
`value`:是否选中，值为true|false,类似checkbox中的checked
`fieldValue`: 类似checkbox中的value
`label`: 选框后面的显示描述

```html
<s:checkbox id="a" name="a" value="true" fieldValue="1" label="A" />
<!-- 转换为checkbox -->
<input type="checkbox" id="a" name="a" value="1" checked>A
```
