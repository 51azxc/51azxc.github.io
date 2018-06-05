title: angular使用过程中碰到的一些小问题解封方法
date: 2016-08-01 16:27:58
tags: ["ui-router", "http"]
categories: ["javascript", "angular"]
---

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

<!-- more -->

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
