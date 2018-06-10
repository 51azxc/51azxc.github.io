title: "Angular零散知识点收集"
date: 2016-08-01 16:27:58
tags: ["Angular", "Angular-Material"]
categories: ["Javascript", "Angular"]
---

收集一些平时使用Angularjs1.x及Angular Material碰到的问题的解决方法.

<!-- more -->

### Angular get方法传参

> [$http get parameters does not work](http://stackoverflow.com/questions/17225088/http-get-parameters-does-not-work)

道在`AngularJS`中使用`POST`方法提交传递参数是:
```js
$http.post('/api/users/add', { user:user })
```
如果是`GET`方法的话则是:
```js
$http.get('/api/users/add', {
	params: { user:user }
})
```

----

### 获取url参数而不刷新页面

> [How can I set a query parameter in AngularJS without doing a route?](http://stackoverflow.com/questions/17513093/how-can-i-set-a-query-parameter-in-angularjs-without-doing-a-route)

当设置了`URL`发生改变时，默认会刷新页面，如果只是想添加查询数据到`url`而无需刷新页面，可以通过`$routeProvider`的属性`reloadOnSearch`来设置
```js
$routeProvider.when('/daylist', {
	templateUrl: 'partials/daylist',
	controller: 'DayListCtrl',
	reloadOnSearch: false
})
```
在`Controller`中就可以使用`$location.search()`方法来获取`url`中的参数，或者设置`query`参数
```
http://localhost:3000/daylist
$location.search('id',123) =>
http://localhost:3000/daylist?id=123
```

----

### 使用`$locationChangeStart`替换`$routeChangeStart`

> [AngularJs - cancel route change event](http://stackoverflow.com/questions/16344223/angularjs-cancel-route-change-event)

在之前我们使用`$routeChangeStart`来触发路由变化之后的操作，如今需要使用`$locationChangeStart`来代替,详见[https://github.com/angular/angular.js/issues/2109](https://github.com/angular/angular.js/issues/2109)
```js
$scope.$on('$locationChangeStart', function(event, next, current){
  $scope.isSignIn = AuthService.isAuthenticated();
  //...
});
```

----

### 处理解析换行符变`<br />`

> [angularjs处理/n转<br/>时候 <br/>不会解析的问题](https://segmentfault.com/q/1010000002891789)
> [angular中的ng-bind-html指令和$sce服务](https://segmentfault.com/a/1190000000639561)

使用`angularjs`时，如果注入的数据中包含了部分特殊字符时，如`\n`要转成`html`标签`<br>`的话，可以使用`ng-bind-html`标签配合`$sce`服务来解决。
```html
<p ng-bind-html="test"></p>
```
然后在`js`代码中使用`$sce`的`trustAsHtml`方法:
```js
app.controller('TestController', ['$scope', '$sce', function($scope,$sce) {
    //将test中的字符串里的换行符转为标签
    $scope.test=$sce.trustAsHtml(str.replace(/\n/g,"<br/>"));
}]);
```

----

### `ui-router`子页面如何获取父级页面的url参数

> [学习 ui-router 系列文章索引](http://bubkoo.com/2014/01/02/angular/ui-router/guide/index/)
> [Angular ui-router - how to access parameters in nested, named view, passed from the parent template?](http://stackoverflow.com/questions/21097820/angular-ui-router-how-to-access-parameters-in-nested-named-view-passed-from)

`ui-router`中如果有子页面嵌套在父级页面，如果需要获取到父级页面传入的路由参数，例如`/users/1/blogs/1`,首先需要在全局配置:
```js
$stateProvider
    .state('user',{
		url: '/users/:userId',
		templateUrl: '/static/partials/user.html',
		controller: 'userCtrl'
	})
	.state('user.blog',{
		url: '/users/:userId/blogs/:blogId',
		templateUrl: '/static/partials/blog.html',
		controller: 'userBlogCtrl'
	})
```
然后在`html`中可以这样调用:
```html
<a ui-sref="user.blog({ userId: blog.author.id, page:1 })">
```
接下来在控制类中通过注入的`$stateParams`获取对应的属性:
```js
app.controller('userBlogCtrl',
	  ['$scope', '$stateParams', '$state', 
		function($scope, $stateParams, $state){
		console.log($stateParams.userId);
		console.log($stateParams.blogId);
	}]);
```

----

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

----

### Angularjs与Requirejs集成

> [使用 RequireJS 加载 AngularJS](http://beginor.github.io/2014/11/17/load-angularjs-with-requirejs.html)
> [Angular — Using RequireJs (AMD)](https://medium.com/angularjs-meetup-south-london/angular-using-requirejs-amd-528358208f84#.ej72m58z6)
> [angularjs集成requirejs](http://www.ddhigh.com/2015/07/development-angularjs-app-with-requirejs/)

首先在页面中引入`requirejs`：
```html
<script src="/static/bower_components/requirejs/require.js" data-main="/static/js/main"></script>
```
这样`requirejs`会自动加载`main.js`,接下来就是配置入口文件`main.js`：
```js
require.config({
    //配置库路径
	paths: {
		'angular': '/static/bower_components/angular/angular',
		'angular-ui-bootstrap': '/static/bower_components/angular-bootstrap/ui-bootstrap-tpls',
		'angular-ui-router': '/static/bower_components/angular-ui-router/release/angular-ui-router'
		//...
	},
    //导出全局变量
	shim: {
		'angular': {
			exports: 'angular'
		},
		'angular-ui-bootstrap': {
			deps: ['angular'],
			exports: 'angular-ui-bootstrap'
		},
		'angular-ui-router': {
			deps: ['angular'],
			exports: 'angular-ui-router'
		}
	    //...
	}
});

//初始化创建ngApp
require(['angular', 'app', 'router', 'controller', 'service', 'directive'], function(angular) {
    angular.bootstrap(document, ['app']);
});
```
接下来是遵循**AMD**规范的`app.js`：
```js
define('app', ['angular', 'angular-ui-bootstrap', 'angular-ui-router'], 
function(angular){
	'use strict';
	return angular.module('app', ['ui.bootstrap', 'ui.router']);
});
```
这样，其他的模块就可以使用`app`了。
接下来是路由管理工具`router.js`:
```js
define(['app'], function(app){
	return app.run(['$rootScope', '$location', '$state', '$stateParams',
		function($rootScope, $location, $state, $stateParams, AuthService, Restangular){
			$rootScope.$on('$stateChangeStart', function(event, toState, toParams, fromState, fromParams){
				if (toState.data.requireLogin) {
					$location.path('/');
				};
			});
		}])
		.config(function($stateProvider, $urlRouterProvider){
			$urlRouterProvider.otherwise('/');
			$stateProvider
				.state('home',{
					url: '/',
					templateUrl: '/static/partials/home.html',
					data: { requireLogin: false }
				});
		});
});
```
在`controller.js`中也可以使用`app`这个根模块了:
```js
define(['app'], function(app) {
    'use strict';
    app.controller('appCtrl',
        ['$scope', '$location', '$http',
        function($scope, $location, $http){
        //...
    }
});
```
其余`service.js`,`directive.js`也类似。