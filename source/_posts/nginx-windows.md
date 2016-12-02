title: "windows下nginx配置"
date: 2015-04-22 23:21:07
tags: "nginx"
categories: "其他"
---

### windows下nginx配置

> [Windows下nginx+tomcat的负载均衡](http://blog.csdn.net/yetaodiao/article/details/23521411)
> [tomcat结合nginx使用小结](http://cxshun.iteye.com/blog/1535188)
> [Nginx - Windows下Nginx基本安装和配置](http://koda.iteye.com/blog/601249)
> [Nginx之虚拟目录-root与alias的区别](http://www.linuxidc.com/Linux/2013-10/92147.htm)

#### 基本配置

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
启动nginx命令: 
```bash
start nginx
```
如果修改了配置文件只想验证有没有出错可以输入以下命令：
```bash
nginx -t
```
修改了配置文件重启的命令为：
```bash
nginx -s reload
```
停止的命令为
```bash
nginx -s stop
```
配置文件conf/nginx.conf
```
#Nginx所用用户和组,window下不指定  
#user  nobody;  
  
#工作的子进程(通常等于CPU数量或者1倍于CPU)  
worker_processes  1;  
  
#错误日志存放路径  
#error_log  logs/error.log;  
#error_log  logs/error.log  notice;  
#error_log  logs/error.log  info;  

#指定pid存放文件  
#pid        logs/nginx.pid;  
  events {  
    #允许最大连接数  
    worker_connections  1024;  
}  
  
  
http {  
    include       mime.types;  
    default_type  application/octet-stream;  
       
     #定义日志格式  
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  
    #                  '$status $body_bytes_sent "$http_referer" '  
    #                  '"$http_user_agent" "$http_x_forwarded_for"';  
  
    #access_log  logs/access.log  main;  
  
    sendfile        on;  
    #tcp_nopush     on;  
  
    #keepalive_timeout  0;  
    keepalive_timeout  65;  
      
    #客户端上传文件大小控制  
    client_max_body_size 8m;  
      
    #gzip  on;  
      upstream localhost {    
                  server localhost:8080;  
                  server localhost:8000;  
         #根据ip计算将请求分配各那个后端tomcat，许多人误认为可以解决session问题，其实并不能。                 
         #同一机器在多网情况下，路由切换，ip可能不同                  
               ip_hash;    
                   }   
                     
    server {  
        listen       9999;  
        server_name  localhost;  
  
        #charset koi8-r;  
  
        #access_log  logs/host.access.log  main;  
  
        location / {  
            root html;  
            index index.html index.htm;  
            #此处的 http://localhost与upstream localhost对应  
            proxy_pass  http://localhost;  
              
            proxy_redirect off;  
            proxy_set_header Host $host;  
            proxy_set_header X-Real-IP $remote_addr;  
            proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;  
            client_max_body_size   10m;   
            client_body_buffer_size  128k;  
            proxy_connect_timeout  100;  
            proxy_send_timeout   100;  
            proxy_read_timeout 100;  
            proxy_buffer_size 4k;  
            proxy_buffers  4 32k;  
            proxy_busy_buffers_size 64k;  
            proxy_temp_file_write_size  64k;  
        }  
  
        #error_page  404              /404.html;  
  
        # redirect server error pages to the static page /50x.html  
        #  
        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   html;  
        }
    }  
```
`server`节点下的配置意义如下:
`listen`: 表示当前的代理服务器监听的端口，默认的是监听80端口。注意，如果我们配置了多个server，这个listen要配置不一样，不然就不能确定转到哪里去了
`server_name`: 表示监听到之后需要转到哪里去
`location`: 表示匹配的路径，这时配置了/表示所有请求都被匹配到这里
`root`: 里面配置了root这时表示当匹配这个请求的路径时，将会在这个文件夹内寻找相应的文件，这里对我们之后的静态文件伺服很有用
`index`: 当没有指定主页时，默认会选择这个指定的文件，它可以有多个，并按顺序来加载，如果第一个不存在，则找第二个，依此类推
`proxy_pass`: 它表示代理路径，相当于转发，而不像之前说的root必须指定一个文件夹
`error_page`: 发生错误时显示的页面

如果需要代理多台服务器，则如上述配置文件里的模块`upstream`中添加相应的服务器访问路径
```
upstream local_tomcat {  
    server localhost:8080;  
    server localhost:9999;  
}
server{  
        location / {  
           proxy_pass http://local_tomcat;  
        }  
        #......其他省略  
} 
```
在server外添加了一个upstream，而直接在`proxy_pass`里面直接用`http://+upstream`的名称来使用。
	我们还是直`接来http://local`host，还是和第一个一样的效果，所有链接都没问题，说明我们配置正确。
`upstream`中的`server`元素必须要注意，不能加`http://`，但`proxy_pass`中必须加

但有时我们就不想它挂的时候访问另外一个，而只是希望一个服务器访问的机会比另外一个大，这个可以在`server`最后加上一个`weight=`数字来指定，数字越大，表明请求到的机会越大
```
upstream local_tomcat {  
    server localhost:8080 weight=1;  
    server localhost:9999 weight=5;  
}
```
这时我们给了jetty一个更高的权值，让它更有机会访问到，实际上当我们刷新`http://localhost`访问的时候发现第二台服务器访问机率大很多，第一台服务器几乎没机会访问，一般情况下，如果我们必须这样用，不要相关太大，以免一个服务器负载太大

----

#### Nginx之虚拟目录-root与alias的区别
1、alias后跟的指定目录是准确的,并且末尾必须加“/”，否则找不到文件
```
location /c/ {
      alias /a/
}
```
如果访问站点`http://location/c`访问的就是`/a/`目录下的站点信息

2、root后跟的指定目录是上级目录，并且该上级目录下要含有和location后指定名称的同名目录才行，末尾“/”加不加无所谓。
```
location /c/ {
      root /a/
}
```
如果访问站点`http://location/c`访问的就是`/a/c`目录下的站点信息。

3、一般情况下，在`location /`中配置`root`，在`location /other`中配置`alias`是一个好习惯。

