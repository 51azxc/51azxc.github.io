title: "Laravel中间件"
date: 2016-01-13 10:47:10
tags:
categories: ["php", "laravel"]
---

> [HTTP 中间件](http://www.golaravel.com/laravel/docs/5.1/middleware/)
> [Laravel 权限控制整理--中间件](http://blog.csdn.net/a437629292/article/details/46120453)

### 生成中间件

使用下列命令创建一个新的中间件：
```bash
php artisan make:middleware LoginMiddleware
```
在项目文件夹下的`app/Http/Middleware`中可以看到新建的中间件，在这个中间件中，指定如果登陆了的用户才能通过验证，反之则跳转到登陆页面,如果进入了不该进入的页面，则跳转到主页面：
```php
<?php

namespace App\Http\Middleware;

use Closure;

class LoginMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $session = $request->session();
		if($session->has('user')){
            //判断是不是进入指定的页面
            if( $request->is('user/*') ){
                return redirect('user');
            }
            return $next($request);
		}else{
			return redirect('login');
		}
    }
}
```
这里的中间件为**前置**中间件,因为其在`$next($request)`做了一系列的动作，如果想在执行请求后完成一些动作，可以生成**后置**中间件，大致的写法为:
```php
public function handle($request, Closure $next)
{
    $response = $next($request);

    // Perform action

    return $response;
}
```

### 配置中间件

#### 全局中间件

如果希望自定义的中间件能够全局执行，则在`app/Http/Kernel.php`中将自定义的中间件完整路径加入到`$middleware`数组中

#### 路由中间件

如果只希望中间件对部分路由起作用，首先需要在`app/Http/Kernel.php`中的`$routeMiddleware`数组添加自定义的中间件:
```php
 protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    //自定义中间件
    'login' => \App\Http\Middleware\LoginMiddleware::class,
];
```
然后便可在`app/Http/routes.php`中对路由指定中间件:
```php
$router->group(['prefix'=>'user','namespace'=>'admin','middleware'=>'login'],function(){
    Route::get('/','UserController@index');
}];
```
也可以使用链式语句指定中间件:
```php
$router->get('/admin',function(){
    //
})->middleware('login');
```
也可以在控制器中指定中间件:
```php
class UserController extends Controller
{
    //控制器实例化方法
    public function __construct()
    {
		//指定控制器中间件
        $this->middleware('auth');
		//对部分方法指定中间件
        $this->middleware('log', ['only' => ['foo', 'bar']]);
		//对部分方法排除中间件
        $this->middleware('subscribed', ['except' => ['foo', 'bar']]);
    }
}

```