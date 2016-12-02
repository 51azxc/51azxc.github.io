title: "Windows下配置Nginx与PHP的开发环境"
date: 2016-01-12 16:31:30
tags: "nginx"
categories: "php"
---

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
