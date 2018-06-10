title: "Angular登陆认证小例子"
date: 2016-08-04 11:30:56
tags: ["Angular"]
categories: ["JavaScript", "Angular"]
---

使用`Angular`来实现登陆跳转等功能的话，其实只要在`ui-router`的`state`中或者`ngRoute`中的`when`方法中添加对应的登陆判断变量，然后在根据相应的路由跳转事件中进行判断即可。

<!-- more -->

> [Handling User Authentication With Angular and Flask](https://realpython.com/blog/python/handling-user-authentication-with-angular-and-flask/)
> [AngularJS User Registration and Login Example & Tutorial](http://jasonwatmore.com/post/2015/03/10/AngularJS-User-Registration-and-Login-Example.aspx)
> [Authentication made simple in Single Page AngularJS Applications](http://brewhouse.io/blog/2014/12/09/authentication-made-simple-in-single-page-angularjs-applications.html)

首先在路由设置中添加对应的变量`requireLogin`来断定是否登陆:
```js
$stateProvider
  .state('home',{
	url: '/',
	templateUrl: '/static/partials/welcome.html',
	controller: 'welcomeCtrl',
	data: { requireLogin: false }
  })
  .state('blog',{
	abstract: true,
	url: '/blogs',
	template: '<ui-view />'
  })
  .state('blog.create',{
	url: '/create',
	templateUrl: '/static/partials/blog.create.html',
	controller: 'blogCreateCtrl',
	data: { requireLogin: true }
  })
  //...
```
接下来在根作用域捕获`$stateChangeStart`事件触发方法中判断:
```js
$rootScope.$on('$stateChangeStart', 
function(event, toState, toParams, fromState, fromParams){
  //如果requireLogin为true且不存在授权信息
  if (toState.data.requireLogin && AuthService.isAuth() === false) {
    //跳转到主页面
    $location.path('/');
    //向子作用域发送消息
    $rootScope.$broadcast('token timeout');
  };
});
```
这里定义了`AuthService`的服务:
```js
myApp.factory('AuthHttpService', ['$q', '$timeout', '$http', function($q, $timeout, $http){
	var sendData = function(url, method, username,password, data){
		var deferred = $q.defer();
		var req = {
			method: method,
			url: url,
			data: data,
			headers: {
				Accept: 'application/json',
				//对数据进行Base64编码
				Authorization: 'Basic ' + btoa(username + ':' + password)
			}
		};
		$timeout(function(){
			$http(req).then(function(response){
				deferred.resolve(response);
			}, function(response){
				deferred.reject(response);
			});
		},1000);
		return deferred.promise;
	};
	return {
		'sendData': sendData
	}
}]);
myApp.factory('AuthService',
  ['$q', 'localStorageService', 'AuthHttpService',
  function($q, localStorageService, AuthHttpService){
  	var login = function(user){
  		var deferred = $q.defer();
  		//访问后端获取数据
		AuthHttpService.sendData('/token', 'POST', user.username, user.password, user)
			.then(function(response){
				if(response.status === 200){
					setToken(response.data);
					deferred.resolve();
				}else{
					deferred.reject();
				}
			},function(response){
				deferred.reject();
			});
  		return deferred.promise;
  	};

  	var logout = function(){
		removeToken();
  	};

    var setToken = function(data){
        将token以及userId存入到本地存储中
		localStorageService.set('token', data.token);
		localStorageService.set('userId', data.userId);
    };

    var getToken = function(){
		return localStorageService.get('token');
    };

    var getUserId = function(){
		return localStorageService.get('userId');
    };

    var removeToken = function(){
		localStorageService.remove('token');
		localStorageService.remove('userId');
    };
    //判断是否已登陆
    var isAuth = function(){
		if (getToken()) {
			return true;
		}else{
			return false;
		}
    };

  	return ({
		isAuth: isAuth,
		login: login,
		logout: logout,
		getToken: getToken,
		getUserId: getUserId
  	});
}]);
```
在根作用域中，我们还需要捕获已经失效的授权，这里使用了`Restangular`插件:
```js
Restangular.addFullRequestInterceptor(function(headers, params, element, httpConfig){
		if(!headers){
		    //获取验证的形式不仅为'用户名:密码'，
		    //也可以为token，因此如果含有token，
		    //则也需要满足'username:password'的格式
			headers = { 'Authorization': 'Basic ' + btoa(AuthService.getToken() + ':unused') };
		}
		headers['Authorization'] = 'Basic ' + btoa(AuthService.getToken() + ':unused');
		return { headers: headers };
	});
	Restangular.setErrorInterceptor(function(response, deferred, responseHandler){
	    //判断是否授权失效
		if (response.status == 401) {
			AuthService.logout();
			$location.path('/');
			alert('token invalid,please sign in');
			$rootScope.$broadcast('token timeout');
			return false;
		}
		return true;
	});
```
在后端使用了`flask`来做授权部分:
```python
from flask.ext.httpauth import HTTPBasicAuth

# flask-httpauth
auth = HTTPBasicAuth()

# 登陆获取token
@app.route('/token', methods=['POST'])
@auth.login_required
def get_auth_token():
    token = g.user.generate_auth_token()
    return jsonify({ 'token': token.decode('ascii'), 'userId': g.user.id })

# 验证，会自动进行base64转码，因此需要将传入的参数以"base64(email_or_token:password)"
# 进行编码
@auth.verify_password
def verify_password(username_or_token, password):
    user = User.verify_auth_token(username_or_token)
    # 不是token登陆则以正常用户名密码登录
    if not user:
        md5 = hashlib.md5()
        md5.update(password.encode('utf-8'))
        hash_password = md5.hexdigest()
        user = User.query.filter_by(username=username_or_token)\
               .filter_by(password=hash_password).first()
        if not user:
            return False
    g.user = user
    return True

@app.route('/blog/create', methods=['POST'])
@auth.login_required
def creaetBlog():
    pass
```
这里面使用了`Flask-HTTPAuth`来进行**HTTP BASIC Authentication**的方法来进行验证。
在标注了`@auth.login_required`的注解的方法则表明需要让`Flask-HTTPAuth`需要验证用户信息。通过实现`verify_password`回调函数去验证用户名和密码，然后`Flask-HTTPAuth`再调用这个回调函数，这样就验证用户是否授权了。
当通过授权的用户信息则是保存在应用上下文`g`对象中，这样其他函数就可以调用到登陆的用户信息，相当于存在`session`中。