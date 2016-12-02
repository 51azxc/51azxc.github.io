title: "Windows下安装Composer"
date: 2016-01-12 15:09:56
tags: ["composer", "symfony"]
categories: "php"
---

> [windows下安装php依赖关系管理工具composer](http://my.oschina.net/u/948242/blog/148269?fromerr=eO6GBWtU)
> [windows下安装composer方法](http://www.tuicool.com/articles/M7F7jyE)
> [解决Win7下运行php Composer出现SSL报错的问题](http://my.oschina.net/yearnfar/blog/346727?fromerr=f5KmKY7s)
> [How to install Symfony 2.7](http://stackoverflow.com/questions/29475044/how-to-install-symfony-2-7)

**Composer**是**PHP**的一个依赖管理工具。
要安装Composer首先需要开启`php_openssl.dll`扩展；开启的方法则是在`php.ini`配置文件中将`extension = php_openssl.dll`这一行前面的分号去掉。
命令行cd到放置Composer文件夹下，使用下列命令：
```bash
php -r "readfile('https://getcomposer.org/installer');" | php
```
或者可以直接[下载](https://getcomposer.org/installer)Composer文件
这里已经完成了Composer的下载工作，可以使用命令`php composer.phar -V`来查看Composer的版本。当然这也稍显麻烦，因此可以在composer.phar文件同级目录下新建一名为`composer.bat`的文件，然后输入下列命令：
```bash
echo @php "%~dp0composer.phar" %*>composer.bat
```
保存之后即可使用`composer -V`来查看当前composer的版本了。
如果想要全局使用，可以将它添加到系统变量*PATH*中。
接下来就可以使用`composer create-project`命令来新建项目，如：
```bash
composer create-project larave/laravel project_name
composer create-project symfony/framework-standard-edition project_name
```

**注意**：如果运行composer出现了SSL错误，有一种可能是没有安装CA证书导致的。首先需要[下载证书](http://curl.haxx.se/docs/caextract.html),然后再修改`php.ini`文件即可
```ini
openssl.cafile= X:/存放证书的路径/cacert.pem
```