title: "Promise处理异步操作"
date: 2016-01-14 15:50:20
tags: "promise"
categories: "nodejs"
---

> [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
> [在Node.js 中用 Q 实现Promise – Callbacks之外的另一种选择](http://www.ituring.com.cn/article/54547)
> [nodejs promise for q.js](http://my.oschina.net/tongjh/blog/275378?fromerr=fB9aMhnD)
> [利用q.js实现node 常用api的promise化](http://www.tuicool.com/articles/AjaUjyJ)
> [q](https://github.com/kriskowal/q)

`javascript`中处理异步一般都是使用回调函数`callback`来解决的。但是这种写法很容易写出金字塔形的代码，可读性差。而是用`promise`链式的代码无疑更容易维护。

首先看一下**ES6**中的`promise`：
```js
function promiseTest() {
	var result = 0;
	console.log(new Date()+' result: '+result);
	var p = new Promise(function(resolve, reject){
	    //模拟耗时操作，1秒后响应
		setTimeout(function(){
			result = Math.round(Math.random() * 1000);
			if(result > 500){
			    //得到返回值
				resolve(result);
			}else{
			    //抛出异常
				reject(result);
			}
		}, 1000);
	});
	p.then(function(val){
		console.log(new Date()+' resolve result: '+result);
	}).catch(function(error){
		console.log(new Date()+' reject result: '+result);
	});
}
```
也可以使用`q.js`来实现`promise`。
首先需要安装`q.js`
```js
npm install q --save
```
然后可以用他来进行改造默认的`callback`风格函数。例如`nodejs`中对文件的读取操作：
```js
var fs = require('fs');

var filename = 'test.txt';
var encoding = 'utf-8';

fs.readFile(filename, encoding, function(err, data){
	if (err) throw err;
	console.log(data)
});
```
* 使用`nfcall`,`nfapply`：
```js
var Q = require('q');

//nfcall与nfapply类似，区别则是传入参数的格式不同
//nfcall的传入参数不定
//nfapply的传入参数为数组
var q_nfcall_test = Q.nfcall(fs.readFile, filename,encoding);
var q_nfapply_test = Q.nfapply(fs.readFile, [filename,encoding]);

q_nfcall_test.then(function(result){
	console.log('q_nfcall_test resolve result: '+result);
},function(error){
	console.log('q_nfcall_test reject result: '+error);
});

q_nfapply_test.then(function(result){
	console.log('q_nfapply_test resolve result: '+result);
},function(error){
	console.log('q_nfapply_test reject result: '+error);
});
```
* 使用`denodeify`:
```js
var q_denodeify_test = Q.denodeify(fs.readFile);
q_denodeify_test(filename, encoding).then(function(result){
	console.log('q_denodeify_test resolve result: '+result);
},function(error){
	console.log('q_denodeify_test reject result: '+error);
});
```
* 使用`makeNodeResolver`:
```js
var q_makeNodeResolver_test = function(filename, encoding){
	var deferred = Q.defer();
	fs.readFile(filename, encoding, deferred.makeNodeResolver());
	return deferred.promise;
};

q_makeNodeResolver_test(filename, encoding).then(function(result){
	console.log('q_makeNodeResolver_test resolve result: '+result);
},function(error){
	console.log('q_makeNodeResolver_test reject result: '+error);
});
```
* 使用`deferd`:
```js
function q_deferred_test(){
	var result = 0;
	var deferred = Q.defer();
	setTimeout(function(){
		result = Math.round(Math.random() * 1000);
		if(result > 500){
			deferred.resolve(result);
		}else{
			deferred.reject(result);
		}
	}, 1000);
	return deferred.promise;
}

q_deferred_test().then(function(result){
	console.log(new Date()+' resolve result: '+result);
},function(error){
	console.log(new Date()+' reject result: '+error);
});
```
可以写出链式操作：
```js
function q_delay_test(value){
	return Q.delay(value, value * 1000);
}

q_delay_test(1).then(function(result){
	console.log(new Date()+' q_delay_test result: '+result);
	//返回值传入下一个promise中
	return q_delay_test(result);
}).then(function(result){
	console.log(new Date()+' q_delay_test result: '+result);
});
```
如果需要并行操作，而返回值各不相干，可以使用`all`方法:
```js
var q_all_test = Q.all([
	q_delay_test(1),
	q_delay_test(2)
]);

q_all_test.then(console.log,console.error);
```
`all`方法返回的返回值为一个数组，数组的值为`all`里面各个函数的返回值。如果需要精确的得到返回值，可以使用`spread`方法：
```js
q_all_test.spread(function(a,b){
	console.log(new Date()+':'+a);
	console.log(new Date()+':'+b);
});
```