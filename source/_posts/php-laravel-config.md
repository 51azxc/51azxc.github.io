title: "Laravel部分配置"
date: 2016-01-12 17:52:50
tags:
categories: ["php", "laravel"]
---

### 开启内置服务

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

### 更改时区

> [Timezones in Laravel 4](https://www.neontsunami.com/posts/laravel-4-timezones)

在`Laravel`项目的congfig文件夹下有个配置文件为`app.php`,找到配置项`timezone`，默认为`UTC`，改成`PRC`即为北京时间