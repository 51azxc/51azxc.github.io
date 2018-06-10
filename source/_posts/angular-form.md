title: "Angular表单部分知识点"
date: 2016-08-03 18:07:14
tags: ["Angular", "form"]
categories: ["Javascript", "Angular"]
---

收集一些使用Angular处理表单的知识点。

<!-- more -->

> [AngularJS Form Validation](https://scotch.io/tutorials/angularjs-form-validation)
> [Form validation with AngularJS](http://www.ng-newsletter.com/posts/form-validation-with-angularjs.html)
> [password-check directive in angularjs](http://stackoverflow.com/questions/14012239/password-check-directive-in-angularjs)

### 基本表单验证

`Angular`的表单验证的部分属性:

| 属性 | css类 | 描述 |
| --- | :---: | --- |
| `$valid` | ng-valid | 表单已经通过了验证，`boolean`类型 |
| `$invalid` | ng-invalid | 表单没有通过验证， `boolean`类型 |
| `$pristine` | ng-pristine | 表单还没有被输入任何字符，`boolean`类型 |
| `$dirty` | ng-dirty | 表单已经有输入字符，清除为空也算， `boolean`类型 |
| `$touched` | ng-touched | 表单的任何输入框处罚了`onblur`事件，即获取焦点后又失去了焦点。`boolean`类型 |

<!-- more -->

接下来开始构建一个表单:
```html
<form name="registerForm" ng-submit="register()" novalidate>
</form>
```
这里指定了表单提交的方法为`register()`,这个方法将在对应的控制器里定义；而`novalidate`则是去除了**HTML5**默认的表单验证。

然后开始往表单里面添加相关表单元素:
```html
<form name="registerForm" ng-submit="register()" novalidate>
  <div class="form-group">
    <input type="text" class="form-control" name="username" placeholder="username" ng-model="user.username" ng-minlength="3" ng-maxlength="12" required>
  </div>
  <div class="form-group">
    <input type="password" class="form-control" ng-model="user.password" name="password" placeholder="password" required>
  </div>
  <div class="form-group">
    <input type="password" class="form-control" ng-model="user.confirmPassword" name="confirmPassword" placeholder="confirm password" required>
  </div>
  <div class="form-group">
    <button class="btn btn-success btn-lg btn-block" type="submit" ng-disabled="registerForm.$invalid || disabled">Register</button>
  </div>
</form>
```
这里添加了三个输入框，分别是：用户名，密码以及确认密码；还有最后的提交按钮。这里面涉及到的关键字有:
* `required`: 必须输入
* `ng-minlength`: 最小输入长度
* `ng-maxlength`: 最大输入长度
* `ng-disabled`: 没通过验证禁止点击提交按钮,这里使用表单的`$invalid`来判断
* `ng-model`: 绑定数据

接下来通过`ng-show`来显示没有通过验证的错误信息
```html
<input type="password" class="form-control" ng-model="user.password" name="password" placeholder="password" required>
<div class="alert alert-warning" ng-show="registerForm.password.$dirty && registerForm.password.$invalid">
  <p ng-show="registerForm.password.$error.required">Password is required</p>
</div>
```
在密码输入框下方添加了警告，通过表单的`$dirty`判断是否有输入过以及`$invalid`是否合法来决定显示警告信息
如果使用了`Bootstrap`来美化界面，还能使用`ng-class`配合其中的`has-error`样式生成警告信息:
```html
<div class="form-group" ng-class="{'has-error': registerForm.password.$invalid && !registerForm.password.$pristine}">
  <input type="password" class="form-control" ng-model="user.password" name="password" placeholder="password" required>
  <div class="alert alert-warning" ng-show="registerForm.password.$dirty && registerForm.password.$invalid">
	<p ng-show="registerForm.password.$error.required">Password is required</p>
  </div>
</div>
```
如果使用了`angular-messages`，可以用`ng-messages`来显示警告信息:
```html
<input type="password" class="form-control" ng-model="user.password" name="password" placeholder="password" required>
<div ng-messages="registerForm.password.$dirty && registerForm.password.$error">
  <div ng-message="required">Password is required.</div>
</div>
```
最后在控制器中是这样的:
```js
myApp.controller('registerCtrl', ['$scope', function($scope){
  $scope.register = function(){
    //判断是不是通过了验证
    if ($scope.registerForm.$valid) {
      $scope.error = false;
      $scope.disabled = true;
      //...
	}
  };
}]);
```

----

### 判断用户名是否重复

接下来需要做的是复杂一点的表单验证。首先需要保证输入的用户名唯一性，因此需要先查询后台数据，如果已经被注册。首先要注册一个指令:
```js
myApp.directive('nameUnique', ['$http', '$timeout', function($http, $timeout){
  var checking = null;
  return {
    require: 'ngModel',
    link: function(scope, elem, attrs, ctrl){
      scope.$watch(attrs.ngModel, function(newVal){
        if(!checking){
          checking = $timeout(function(){
            $http.post('/check', {'name': newVal}).then( function(response){
              ctrl.$setValidity('unique', response.data.isUnique);
              checking = null;
            }, function(error){
            checking = null;
            });
          }, 500);
        }
      });
    }
  };
}]);
```
这里注册了一个名为`nameUnique`的指令，引入了`$http`,`$timeout`服务。返回的参数中,`require`表明这个指令需要在元素中搜索`controller`,并且放入到`link`函数的参数中。
`link`函数则是当`angular`读取了指令之后，完成了`compile`将指令转成对应的`HTML`元素之后所进行的一系列操作，例如数据的双向绑定等等。在`link`函数中，传入的参数分别是`scope`：作用域；`elem`:对应的元素；`attrs`:元素中的所有属性（以`map`的形式返回）以及上文中提到的`controller`。
`link`函数中调用了`$timeout`方法，延迟0.5s运行，防止对后端频繁请求过大，作用于`scope`通过`$watch`方法来监听元素属性`ngModal`对应的值，如果有变化就进行下面的函数操作，请求后端，如果是唯一的，则通过`$setValidity`方法设置其在表单中的有效性。

在`HTML`中这样调用:
```html
<input type="text" class="form-control" name="username" placeholder="username" ng-model="user.username" name-unique="username" required>
<div class="alert alert-warning" ng-show="registerForm.username.$dirty && registerForm.username.$invalid">
  <p ng-show="registerForm.username.$error.required">Username is required</p>
  <p ng-show="registerForm.username.$error.unique">Username is taken, try another one</p>
</div>
```
在这里使用的新指令为`name-unique`，也可以使用`HTML5`新规范`x-`，`x-data`前缀，解析时会把前缀去掉。由于`HTML`对标签的大小写不敏感，因此需要通过连接符`-`或者`:`来使用驼峰命名法的指令。

----

### 密码一致性

接下来需要判断的则是密码确认一致性。这里也需要新建立一个指令:
```js
myApp.directive('passwordMatch', [function(){
  return {
    restrict: 'A',
    scope: { 
      passwordMatch: '=' 
    },
    require: 'ngModel',
    link: function(scope, elem, attrs, ctrl){
      scope.$watch(function(){
        var combined;
        if(scope.passwordMatch || ctrl.$viewValue){
          combined = scope.passwordMatch + '_' + ctrl.$viewValue;
        }
        return combined;
      }, function(value){
        if(value){
        /**
         * This function is added to the list of the $parsers.
         * It will be executed the DOM (the view value) change.
         * Array.unshift() put it in the beginning of the list, so
         * it will be executed before all the other
         */
        ctrl.$parsers.unshift(function(viewValue){
        var origin = scope.passwordMatch;
        if (origin != viewValue){
          ctrl.$setValidity('match', false);
          return undefined;
        }else{
          ctrl.$setValidity('match', true);
          return viewValue;
        }
      });
    }
  });
  }
  };
}]);
```
部分属性解释：
* `restrict`: 指定这个指定该如何使用。通常有**A**表示为*属性*,**E**表示为*元素*,**C**表示为`class`。
* `scope`: 指定作用域。默认情况下不指定是使用父级作用域，上述指定则是对于`passwordMatch`这个属性指定的值双向绑定。在监控整个属性的同时，还能够改变父级作用域的此属性的值。
 
接下来还是`link`函数，在`$watch`中，第一个参数则为需要监控的数值，这里我们读取了作用域`passwordMatch`指定的值以及控制器中页面可见元素数值`$viewValue`组合起来。
`$parsers`是一个需要执行的函数的列表。当控制器读取了`DOM`的值时，里面的函数会按照顺序执行。这里将最先执行匹配方法。失败的话则返回`undefined`,表示解析发生错误，将不会运行`ngModel`的`$validators`。关于这部分可以参考[官方文档](https://docs.angularjs.org/api/ng/type/ngModel.NgModelController)

接下来在`HTML`中的使用新指令:
```html
<input type="password" ng-model="user.password" name="password" password-match="user.confirmPassword" required>
<div ng-messages="registerForm.password.$dirty && registerForm.password.$error">
  <div ng-message="required">This is required.</div>
  <div ng-message="match">Password is not match</div>
</div>

<input type="password" ng-model="user.confirmPassword" name="confirmPassword" password-match="user.password" required>
<div ng-messages="registerForm.confirmPassword.$dirty && registerForm.confirmPassword.$error">
  <div ng-message="required">This is required.</div>
  <div ng-message="match">Password is not match</div>
</div>
```
关于**指令**部分，可以参考[官方文档](https://docs.angularjs.org/guide/directive)
