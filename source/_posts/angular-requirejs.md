title: angularjs与requirejs集成
date: 2016-08-04 18:15:17
tags: ["requirejs"]
categories: ["javascript", "angular"]
---

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


