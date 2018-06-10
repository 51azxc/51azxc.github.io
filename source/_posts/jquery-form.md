title: "jQuery 部分插件整理"
date: 2015-05-12 09:47:39
tags: ["jQuery", "form", "table"]
categories: ["javascript", "jQuery"]
---

收集一些jQuery插件使用过程中碰到的问题解决方法。

<!-- more -->

### jQuery Form表单部分

#### jquery 提交中文问题
> 来自[jquery解决提交中文问题两种方法](http://hi.baidu.com/computercode/item/84fb7b12efa0ab0be75c3602)

* 方法一
```js
//客户端页面
var name = encodeURI($("#name").val());
//服务端
String name= new String(request.getParameter("name").getBytes("ISO-8859-1"),"UTF-8");
```

* 方法二
```js
var name = encodeURI(encodeURI($("#name").val()));
//服务端
String name= request.getParameter("name");
try{
    name= URLDecoder.decode(name,"UTF-8");
}catch(Exception e){}
```

----------

#### jquery 判断radio是否选中

> [JQuery判断radio是否选中，获取选中值](http://www.cnblogs.com/xcj1989/archive/2011/06/29/jquery_radio.html)

```js
var val = $("input:radio[name='radio']:checked").val();
if(val==null){
    alert("什么也没选中!");
}
```

----------

#### jquery对checkbox的操作

> [JQuery对CheckBox的一些相关操作](http://for-dream.iteye.com/blog/1570434)
> [设置checkbox复选框为readonly只读的两种方式](http://www.jbxue.com/article/12668.html)
> [jquery判断checkbox是否被选中](http://www.cnblogs.com/starweb/archive/2008/10/13/1309670.html)

获取checkbox选中的值
```js
var arr = new Array();
$("input[name='checkbox_name']:checked").each(function(){
  arr.push($(this).val());
});
//输出选中的值
console.log(arr.join(","));
```

设置checkbox为只读
```html
<input type="checkbox" onclick="return false">
```
或者使用js
```js
checkbox.onclick = function(){ return false; }
```
或者使用jquery的click方法
```js
$("#checkbox_id").click(function(){
  this.checked = !this.checked;
});
```

判断checkbox是否被选中
```js
if($("#checkbox_id").is(":checked"))
if($("#checkbox_id").prop("checked"))
```

----------

#### jquery对select操作

> [JQuery设置select控件只读](http://nvry.iteye.com/blog/1556157)
> [jquery获得select option的值 和对select option的操作](http://www.cnblogs.com/QQJnet/archive/2011/12/11/2284174.html)
> [高手指导下 jquery 中 select 控件 ，根据text值来选中项没用](http://bbs.csdn.net/topics/390140793)

常用操作
```js
//获取Select选择的Text
var txt = $("#select_id").find("option:selected").text();
//获取Select选择的Value 
var val = $("#select_id").val();
//获取Select选择的索引值 
var index = $("#select_id ").get(0).selectedIndex;
//获取Select最大的索引值 
var maxIndex=$("#select_id option:last").attr("index");

//设置Select索引值为1的项选中
$("#select_id ").get(0).selectedIndex=1;
// 设置Select的Value值为1的项选中
$("#select_id ").val(1);
//设置Select的Text值为jQuery的项选中
$("#select_id option[text='jQuery']").attr("selected", true);

//为Select追加一个Option(下拉项)
$("#select_id").append("<option value='Value'>Text</option>");
/删除Select中索引值最大Option(最后一个) 
$("#select_id option:last").remove();
//删除Select中Text='4'的Option 
$("#select_id option[text='4']").remove();

//清空选项
$("#select_id").empty();
```

设置select为只读
```js
$(function(){
  $("#select_id").focus(function(){
    $(this).attr('defaultIndex',$(this).attr('selectedIndex'));
  });
  $("#select_id").change(function(){
    $(this).attr('selectedIndex',$(this).attr('defaultIndex')); 
  });
});
```

----------

#### jquery.form插件

> [jquery表单验证插件 jquery.form.js](http://www.cnblogs.com/luluping/archive/2009/04/15/1436177.html)
> [jQuery插件 -- Form表单插件jquery.form.js](http://blog.csdn.net/zzq58157383/article/details/7718956)
> [使用jQuery.form插件，实现完美的表单异步提交](http://www.cnblogs.com/heyuquan/p/form-plug-async-submit.html)

核心方法 -- `ajaxForm()` 和 `ajaxSubmit()`
```js
$('#myForm').ajaxForm(function() {     
   $('#output1').html("提交成功！欢迎下次再来！").show();      
});    
         
$('#myForm2').submit(function() {  
   $(this).ajaxSubmit(function() {     
      $('#output2').html("提交成功！欢迎下次再来！").show();      
   });  
   return false; //阻止表单默认提交  
});
```
ajaxForm() 和 ajaxSubmit() 都能接受0个或1个参数，当为单个参数时，该参数既可以是一个回调函数，也可以是一个options对象，上面的例子就是回调函数，下面介绍options对象，使得它们对表单拥有更多的控制权
```js
var options = {  
   target: '#output',          //把服务器返回的内容放入id为output的元素中      
   beforeSubmit: showRequest,  //提交前的回调函数  
   success: showResponse,      //提交后的回调函数  
   //url: url,                 //默认是form的action， 如果申明，则会覆盖  
   //type: type,               //默认是form的method（get or post），如果申明，则会覆盖  
   //dataType: null,           //html(默认), xml, script, json...接受服务端返回的类型  
   //clearForm: true,          //成功提交后，清除所有表单元素的值  
   //resetForm: true,          //成功提交后，重置所有表单元素的值  
   timeout: 3000               //限制请求的时间，当请求大于3秒后，跳出请求  
}  
  
function showRequest(formData, jqForm, options){  
   //formData: 数组对象，提交表单时，Form插件会以Ajax方式自动提交这些数据，格式如：[{name:user,value:val },{name:pwd,value:pwd}]  
   //jqForm:   jQuery对象，封装了表单的元素     
   //options:  options对象  
   var queryString = $.param(formData);   //name=1&address=2  
   var formElement = jqForm[0];              //将jqForm转换为DOM对象  
   var address = formElement.address.value;  //访问jqForm的DOM元素  
   return true;  //只要不返回false，表单都会提交,在这里可以对表单元素进行验证  
};  
  
function showResponse(responseText, statusText){  
   //dataType=xml  
   var name = $('name', responseXML).text();  
   var address = $('address', responseXML).text();  
   $("#xmlout").html(name + "  " + address);  
   //dataType=json  
   $("#jsonout").html(data.name + "  " + data.address);  
};  
  
$("#myForm").ajaxForm(options);  
  
$("#myForm2").submit(funtion(){  
   $(this).ajaxSubmit(options);  
   return false;   //阻止表单默认提交  
});
```
表单提交之前进行验证：  beforeSubmit会在表单提交前被调用，如果beforeSubmit返回false，则会阻止表单提交
```js
beforeSubmit: validate  
function validate(formData, jqForm, options) { //在这里对表单进行验证，如果不符合规则，将返回false来阻止表单提交，直到符合规则为止  
   //方式一：利用formData参数  
   for (var i=0; i < formData.length; i++) {  
       if (!formData[i].value) {  
            alert('用户名,地址和自我介绍都不能为空!');  
            return false;  
        }  
    }   
  
   //方式二：利用jqForm对象  
   var form = jqForm[0]; //把表单转化为dom对象  
      if (!form.name.value || !form.address.value) {  
            alert('用户名和地址不能为空，自我介绍可以为空！');  
            return false;  
      }  
  
   //方式三：利用fieldValue()方法，fieldValue 是表单插件的一个方法，它能找出表单中的元素的值，返回一个集合。  
   var usernameValue = $('input[name=name]').fieldValue();  
   var addressValue = $('input[name=address]').fieldValue();  
   if (!usernameValue[0] || !addressValue[0]) {  
      alert('用户名和地址不能为空，自我介绍可以为空！');  
      return false;  
   }  
  
    var queryString = $.param(formData); //组装数据  
    //alert(queryString); //类似 ： name=1&add=2    
    return true;  
}
```

----

### dataTables插件

> [dataTables-使用详细说明整理](http://blog.csdn.net/mickey_miki/article/details/8240477)
> [datatables request unknown parameter '0' from data source for row 0](http://blog.csdn.net/badboyer/article/details/8509484)
> [Jquery datatable 配置与应用](http://www.cnblogs.com/kezf/p/datatable.html)
> [fnFilter](http://datatables.net/forums/discussion/537/fnfilter/p1)
> [Jquery DataTables 自定义布局sdom](http://blog.csdn.net/bill1315/article/details/12577595)
> [jQuery DataTables插件 单个/批量设置列的显示/隐藏状态](http://lyj86.iteye.com/blog/1824787)
> [jquery datatables adding row_selected class to a row](http://stackoverflow.com/questions/5454805/jquery-datatables-adding-row-selected-class-to-a-row)
> [求高手指点jquery 的datatables插件问题](http://bbs.csdn.net/topics/370191615)
> [How to create JQuery DataTable using JSON and servlet](http://techmytalk.com/2013/08/24/how-to-create-jquery-data-table-using-json-pass-by-servlet/)

[datatables](http://www.datatables.net)是jquery的一款表格插件,接收后端传送过来的json格式的数据把他显示在页面table中，首先对于页面中，必须要有一个空的table，并且保证拥有表头元素:
```html
<table id="list">
  <thead>
    <th>head1</th>
    <th>head2</th>
    <th>...</th>
  </theand>
</table>
```
这里的表头个数必须与后端传过来的数组长度对应，不然会报错。
接下来是datatables的部分配置:
```js
$('#list').dataTable({
    //当载入的时候显示进度条
	"bProcessing": true,
	//从服务端获取数据
	"bServerSide": true,
	//服务端链接
	"sAjaxSource": "list",
	//分页显示数组
	//"sPaginationType": "full_numbers",
	//使用jQueryUI主题
	"bJQueryUI": false,
	//客户端过滤框
	"bFilter": false,
	"bSortClasses": false,
	//指定排序的列
	"aaSorting": [[2, 'desc']],
	//如果后端传入的不是数组而是map，则需要指定列对应的key值
    /*"aoColumns":[{"mDataProp":"name"},
	             {"mDataProp":"age"},
				 {"mDataProp":"id"}],*/
	//渲染列，可以指定列是否排序，是否可见，宽度等
	"aoColumnDefs": [{"bSortable":false, "aTargets":[0]},
	                 {"bVisible":false, "aTargets":[1]},
	                 {"sWidth":"160px", "aTargets":[2]}],
	"fnRowCallback":function(nRow, aData, iDisplayIndex){
	    //这里对行进行渲染操作，可以对行数据进行修改
		$('td:last',nRow).html("test");
	},
	"fnServerData":function(sSource, aoData, fnCallback){
		//这里是去后端获取数据，可以加入条件查询的参数
		$.getJSON(sSource,aoData,function(json){ fnCallback(json); });
	},
	"oLanguage": {
	    "sProcessing": "正在加载中......",
        "sLengthMenu": "每页显示 _MENU_ 条记录",
        "sZeroRecords": "对不起，查询不到相关数据！",
        "sEmptyTable": "表中无数据存在！",
        "sInfo": "当前显示 _START_ 到 _END_ 条，共 _TOTAL_ 条记录",
        "sInfoFiltered": "数据表中共为 _MAX_ 条记录",
        "sSearch": "搜索",
        "oPaginate": {
            "sFirst": "首页",
            "sPrevious": "上一页",
            "sNext": "下一页",
            "sLast": "末页"
        }
	    //支持通过文本链接获取语言支持
		//"sUrl": "resources/datatables/zh_CN.txt"
    }
});
```
使用`aTable.fnDraw()`方法可以刷新视图,当向后端请求数据时，datatables向后端传入的请求部分参数有
`sEcho`:请求标识符，需直接返回
`iSortCol_0`: 需排序列的索引,即`"aaSorting": [[2, 'desc']]`中的2
`sSortDir_0`: 升降序排序标识，为asc/desc
`iDisplayStart`: 分页显示开始页面
`iDisplayLength`: 分页长度
返回的数据为json格式数据，对应参数为
```java
Map m = new HashMap();
m.put("sEcho", sEcho);
//总共显示的记录数
m.put("iTotalRecords", count);
//条件查询后的记录数
m.put("iTotalDisplayRecords", filterCount);
//数据数组
m.put("aaData", aaData);
//转换为json格式
return gson.toJson(m);
```
