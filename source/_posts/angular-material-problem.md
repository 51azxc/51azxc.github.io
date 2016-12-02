title: angular material部分问题
date: 2016-08-01 15:35:22
tags: "material"
categories: ["javascript", "angular"]
---

### Angular Material更改日期选择组件默认格式

> [Change format of md-datepicker in Angular Material](http://stackoverflow.com/questions/32566416/change-format-of-md-datepicker-in-angular-material)

`Angular Material`中的日期选择组件`$mdDateLocaleProvider`选中的日期格式为`dd/MM/yyyy`,如果想要替换成其他格式(比如`yyyy-MM-dd`)，可以使用`formatDate`方法:
```js
myApp.config(['$mdDateLocaleProvider', function($mdDateLocaleProvider) {
  $mdDateLocaleProvider.formatDate = function(date) {
		if (date instanceof Date && !isNaN(date.valueOf())) {
			var y = date.getFullYear().toString();
			var m = (date.getMonth()+1).toString();
			var d = date.getDate().toString();
			return y + '-' + (m[1]?m:"0"+m) + '-' + (d[1]?d:"0"+d);
		}
		return '';
  };
}]);
```

----

### Angular Material对话框传值

> [Passing data to mdDialog](http://stackoverflow.com/questions/31240772/passing-data-to-mddialog)

`$mdDialog`中有个属性是`locals`,通过它可以给对应的控制器传入参数:
```js
$mdDialog.show({
  templateUrl: 'showImg.html',
  locals: { imgUrl: url },
  controller: function($scope, $mdDialog, imgUrl){
    $scope.imgUrl = imgUrl;
  	$scope.cancel = function(){
  	  $mdDialog.cancel();
  	};
  },
  clickOutsideToClose:true,
  targetEvent: ev
});
```
传入的参数必须以对象形式传入，必须从控制器中注入
