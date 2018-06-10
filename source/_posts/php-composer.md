title: "PHP配置相关知识点收集"
date: 2016-01-12 15:09:56
tags: ["composer"]
categories: "PHP"
---

### Windows下安装Composer

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

----

### Windows下配置Nginx与PHP的开发环境

> [转载:配置Windows下Nginx + PHP 开发环境](http://www.cnblogs.com/naniannayue/archive/2010/08/07/1794525.html)
> [Windows下配置nginx+php(wnmp)](http://www.cnblogs.com/wuzhenbo/p/3493518.html)
> [windows安装nginx跑php 再加上Laravel](http://blog.csdn.net/yuliyige/article/details/44471787)

### PHP部分配置

修改`php.ini-development`文件:

- 将`extension_dir`前面的分号去掉，并将值改为php文件夹内ext文件夹的路径，如`extension_dir = "C:/mine/php/ext"`
- 将`enable_dl`前面的分号去掉，并将值改为`On`，如`enable_dl = On`
- 将`cgi.force_redirect`前面的分号去掉，并将值改为0，如`cgi.force_redirect = 0`
- 将`fastcgi.impersonate`前面的分号去掉
- 将`cgi.rfc2616_headers`前面的分号去掉，并将值改为1，如`cgi.rfc2616_headers = 1`
- 将`date.timezone`前面的分号去掉，并将值改成Asia/Shanghai,如`date.timezone = Asia/Shanghai`
- 将`extension=php_mysql.dll`,`extension=php_mysqli.dll`,`extension=php_pdo_mysql.dll`前面的分号去掉，以支持MySQL数据库(可选)
- 将`extension=php_curl.dll`前面的分号去掉，以开启`curl`扩展(可选)
- 将`extension=php_openssl.dll`前面的分号去掉，以下载`composer`(可选)

修改完毕后另存为`php.ini`文件至**php**根目录下。

----

### Nginx部分配置

修改**nginx**的配置文件`nginx.conf`,以支持PHP:
```
location ~ \.php$ {
    root           C:/workspace/php/laraveltest2/public;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

添加一个新的路径:
```
location / {
    root   C:/workspace/php/laravelyh/public;
    index  index.html index.htm index.php;
	try_files $uri $uri/ /index.php?$query_string;
}
```

----

### 运行

在命令行输入命令
```bash
C:/mine/php/php-cgi.exe -b 127.0.0.1:9000 -c C:/mine/php/php.ini
```
然后再另开一个命令行窗口，定位到nginx目录下，启动nginx服务
```bash
start nginx
```

接着在浏览器输入`http://localhost`查看效果
