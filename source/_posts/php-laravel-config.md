title: "Laravel部分知识点收集"
date: 2016-01-12 17:52:50
tags: ["Laravel"]
categories: ["PHP"]
---

### Laravel开启内置服务

> [Running PHP's Built-in Web Server](http://laravel-recipes.com/recipes/282/running-phps-built-in-web-server)

`Laravel`内置了一个Web服务器，当需要调试或者查看效果时，可以直接使用命令开启服务:
```bash
php artisan serve
```
默认端口为8000，也可以指定端口:
```bash
php artisan serve --port=8888
```

----

### Laravel更改时区

> [Timezones in Laravel 4](https://www.neontsunami.com/posts/laravel-4-timezones)

在`Laravel`项目的congfig文件夹下有个配置文件为`app.php`,找到配置项`timezone`，默认为`UTC`，改成`PRC`即为北京时间

----

### Laravel中间件

> [HTTP 中间件](http://www.golaravel.com/laravel/docs/5.1/middleware/)
> [Laravel 权限控制整理--中间件](http://blog.csdn.net/a437629292/article/details/46120453)

#### 生成中间件

使用下列命令创建一个新的中间件：
```bash
php artisan make:middleware LoginMiddleware
```
在项目文件夹下的`app/Http/Middleware`中可以看到新建的中间件，在这个中间件中，指定如果登陆了的用户才能通过验证，反之则跳转到登陆页面,如果进入了不该进入的页面，则跳转到主页面：

<!-- more -->

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

#### 配置中间件

##### 全局中间件

如果希望自定义的中间件能够全局执行，则在`app/Http/Kernel.php`中将自定义的中间件完整路径加入到`$middleware`数组中

##### 路由中间件

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

----

### Laravel ORM

#### 获取id

> [Data van from database = ErrorException Undefined property](http://stackoverflow.com/questions/23009678/data-van-from-database-errorexception-undefined-property)

默认的`get()`方法返回的是一个记录的集合，如果需要得到一条记录的id，可以使用如下方法：
```php
$this->select('id')->where('property', '=', property)->first()->id;
```
或者
```php
$this->select('id')->where('property', '=', property)->firstOrFail();
```

----

#### 级联删除

> [Automatically deleting related rows in Laravel (Eloquent ORM)](http://stackoverflow.com/questions/14174070/automatically-deleting-related-rows-in-laravel-eloquent-orm)

在一对多的两个类的关系中，当一的一方被删除了之后，多的一方应该全部被删除
```php
class User extends Eloquent
{
    public function photos()
    {
        return $this->has_many('Photo');
    }

    // this is a recommended way to declare event handlers
    protected static function boot() {
        parent::boot();

        static::deleting(function($user) { // before delete() method call this
             $user->photos()->delete();
             // do the rest of the cleanup...
        });
    }
}
```