title: "php基本知识补遗"
date: 2016-01-11 17:51:03
tags: ["curl", "closure"]
categories: "php"
---

### 发送HTTP请求

> [通过http post发送json数据](http://blog.csdn.net/zhulei632/article/details/8014921)
> [PHP扩展CURL的用法详解](http://www.jb51.net/article/51299.htm)
> [PHP中使用cURL实现Get和Post请求的方法](http://www.jb51.net/article/34745.htm)
> [php发送http请求](http://www.cnblogs.com/simpman/p/3549816.html)
> [PHP curl CURLOPT_RETURNTRANSFER参数的作用使用实例](http://www.jb51.net/article/60855.htm)
> [PHP中CURL方法curl_setopt()函数的参数](http://www.cnblogs.com/txw1958/archive/2013/01/19/2867584.html)

**PHP**中发送*HTTP*请求可以使用`curl`或者`file_get_contents()`方法，`curl`使用前需要在`php.ini`中将`extension=php_curl.dll`前面的分号去掉

<!-- more -->

```php
<?php

$url = 'http://localhost:5000/hello';
$data = json_encode(array('name'=>'tom'));

printf("test1=>");
function sendData1($url, $data) {
	//初始化curl
	$ch = curl_init();
    //请求的url
    curl_setopt($ch, CURLOPT_URL, $url);
    //结果是否直接输出，1为保存到变量不输出
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
	//设置请求头
	curl_setopt($ch, CURLOPT_HTTPHEADER, array(
		'Content-Type: application/json; charset=utf-8',
		'Content-Length: ' . strlen($data)
	));
	//设置请求方式为post
	curl_setopt($ch, CURLOPT_POST, 1);
    //设置post的数据
    curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
    //运行curl，得到返回结果
    $result = curl_exec($ch);
    //关闭curl连接
    curl_close($ch);
	
	return $result;
}
//json_decode的第二个参数为true时，返回的对象为array而不是object,默认为false
var_dump(json_decode(sendData1($url, $data), true));

print_r("test2=>");
function sendData2($url, $data) {
	$options = [
		'http'=>[
			'method'=>'POST',
			//设定类型为json类型
			'header'=>'Content-Type: application/json',
			'content'=>$data,
			//超时时间
			'timeout'=>15*60
		]
	];
	$context = stream_context_create($options);
	$result = file_get_contents($url, false, $context);
    return $result;
}
var_dump(json_decode(sendData2($url, $data)));
?>
```
通过命令行运行命令`php filename.php`即可
响应的服务端为**Python**的*Flask*实现
```php
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello World!'

@app.route('/hello', methods=['POST'])
def hello():
    name = request.json['name'] if request.json['name'] else ''
    return jsonify({'message': 'Hello '+name})

if __name__ == '__main__':
    app.run(host='0.0.0.0',debug=True)

```

----

### 去除Object中的部分属性

> [Is it possible to delete an object's property in PHP?](http://stackoverflow.com/questions/3600750/is-it-possible-to-delete-an-objects-property-in-php)

使用`unset()`方法即可去除指定**Ojbect**中的指定属性
```php
$a = new stdClass();

$a->new_property = 'foo';
var_export($a);  // -> stdClass::__set_state(array('new_property' => 'foo'))

unset($a->new_property);
var_export($a);  // -> stdClass::__set_state(array())
```

----

### 闭包

> [匿名函数](http://php.net/manual/zh/functions.anonymous.php)
> [Closure 类](http://php.net/manual/zh/class.closure.php)
> [PHP的闭包](http://www.cnblogs.com/yjf512/archive/2012/10/29/2744702.html)
> [PHP中的闭包详解](http://www.tuicool.com/articles/FFZbay)
> [条件组合查询问题](http://wenda.golaravel.com/question/367)
> [Eloquent ORM关于拆分查询chunk](http://wenda.golaravel.com/question/358)

闭包即为匿名函数。创建一个没有名称的函数，使用`use`关键字来连接闭包与外界的变量，使子函数可以使用父函数的局部变量
```php
$num = 1;
function test1($num) {
    $result = 1;
	$a = function() use ($num, &$result){
		$result += $num;
		print("result in a: ".$result."\n");
    };
    $a();
	print($result);
    print("\n");
    //不加&则不会影响父函数的局部变量
    $b = function() use ($num, $result) {
		$result -= $num;
		print("result in b: ".$result."\n");
    };
    $b();
	print($result);
}

test1($num);
```