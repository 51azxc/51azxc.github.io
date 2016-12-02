title: "displayTag 属性"
date: 2015-04-21 23:30:16
tags: "displayTag"
categories: "web"
---

> [`<display:setProperty name="basic.msg.empty_list_row" value=""/>` 属性](http://hi.baidu.com/lxiangshanyu/item/014d6f2609e08a3b94f62b8f)

### 更改空记录提示语言，默认提示：“没有满足查询条件的记录”
在`<display:table....`里面 `<display:column...`.前面增加
```html
<display:setProperty name="basic.msg.empty_list_row" value="没有找到任何记录" />
<display:setProperty name="basic.msg.empty_list" value="没有找到任何记录！" />
```
空记录提示语言修改为："没有找到任何记录"

### 空记录的时候仍然显示表头信息（默认空记录不显示表头信息）
在`<display:table....`里面 `<display:column....`前面增加
```html
<display:setProperty name="basic.msg.empty_list_row" value="" />
<display:setProperty name="basic.empty.showtable" value="true" />
```

### 不显示标题，默认为显示=true
```html
<display:setProperty name="basic.show.header" value="false" />
```
### 按list或page排序，默认按page
```html
<display:setProperty name="sort.amount" value="list" /> 
```