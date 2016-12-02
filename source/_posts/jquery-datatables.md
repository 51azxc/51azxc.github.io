title: "jquery dataTables插件"
date: 2015-05-12 16:04:11
tags: "datatables"
categories: ["javascript", "jQuery"]
---

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
