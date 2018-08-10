title: "Node.js部分框架知识点整理"
date: 2016-04-23 22:56:28
tags: ["Express", "Mongoose", "Multer", "Cheerio"]
categories: ["Node.js", "Express"]
---

使用Express框架及周边的一些框架碰到了一系列的问题，把解决方法整理记录一下。

<!-- more -->

### 解决multer无法识别ng-file-upload批量上传文件

> [Multer not accepting files in array format gives 'Unexpected Filed Error'](http://stackoverflow.com/questions/32917617/multer-not-accepting-files-in-array-format-gives-unexpected-filed-error)

`ng-file-upload`上传插件批量上传文件时默认使用`files[0]`, `files[1]`, `files[2]`...这样的数组形式标识上传文件，而`multer`无法识别同样名字的上传文件。因此需要在`ng-file-upload`的配置中修改**arrayKey**属性：
```js
Upload.upload({
  url: '/upload',
  arrayKey: '', // default is '[i]'
  data: {
    files: files
  }
})
```

----

### Express中替换模板为HTML

> [Why is express telling me that my default view engine is not defined?](http://stackoverflow.com/questions/17560760/why-is-express-telling-me-that-my-default-view-engine-is-not-defined)

**Express4**中的默认模板为*Jade*，如果要替换为*html*则需要引入*ejs*模板再指定模板文件：
```js
var ejs = require('ejs');
app.set('view engine', 'html');
app.engine('html', ejs.renderFile);
```

----

### Mongoose 部分知识点

#### 加密字段

> [Mongoose password hashing](http://stackoverflow.com/questions/14588032/mongoose-password-hashing)
> [Mongoose - validate email syntax](http://stackoverflow.com/questions/18022365/mongoose-validate-email-syntax)

存储如用户密码相关字段时，需要对数据进行加密，这里使用最简单的md5加密方法:
```js
var UserSchema = new Schema({
	email: { type: String, trim: true, required: true, unique: true},
	password:  { type: String, required: true, set: hashPassword }
});
function hashPassword(password){
	var md5 = crypto.createHash('md5');
	md5.update(password);
	return md5.digest('hex');
}

//验证密码
UserSchema.methods.comparePassword = function(candidatePassword){
	return this.password === hashPassword(candidatePassword);
};
//验证邮箱
UserSchema.path('email').validate(function (email) {
   var emailRegex = /^([\w-\.]+@([\w-]+\.)+[\w-]{2,4})?$/;
   return emailRegex.test(email);
}, 'Invalid email address');
```

----

#### 关联ID

> [unable to use $match operator for mongodb/mongoose aggregation with ObjectId](http://stackoverflow.com/questions/16310598/unable-to-use-match-operator-for-mongodb-mongoose-aggregation-with-objectid)

虽然`mongodb`是*nosql*类型的数据库，如果想类似与sql类型的数据库对两张表进行关联的话可以使用内置的`ObjectId`字段关联，在建立表关联的时候需要将字段设置成`mongoose.Schema.Types.ObjectId`类型，进行`$match`查询时需要转换为`mongoose.Types.ObjectId`类型,如:
```js
var UserSchema = new Schema({
	email: { type: String, trim: true, required: true, unique: true},
	password:  { type: String, required: true, set: hashPassword },
	bills: [{ type: Schema.Types.ObjectId, ref: 'Bill' }]
});

//select
$matchObject['host'] = mongoose.Types.ObjectId(hostId);
var condition = [{ $match: $matchObject }];
```

----

#### 转换时区

> [Mongodb aggregate timezone 问题](https://gitsea.com/2014/07/26/mongodb-aggregate-timezone-%E9%97%AE%E9%A2%98/)

**Mongodb**默认存入的时间为UTC时间，因此进行聚合操作的时候需要对时间进行时区转换:
```js
var timeOffset = () => {
  //The getTimezoneOffset() method returns the time difference between UTC time and local time, in minutes.
  var offset = new Date().getTimezoneOffset();
  offset = -offset * 60 * 1000;
  return offset;
};

$group: {
  '_id': {
    'year': { $year: { $add: ['$costDate', utils.timeOffset] } },
    'month': { $month: { $add: ['$costDate', utils.timeOffset] } },
    'day': { $dayOfMonth: { $add: ['$costDate', utils.timeOffset] } }
  }
}
```

----

### NodeJs网页爬虫

> [request/request](https://github.com/request/request)
> [cheeriojs/cheerio](https://github.com/cheeriojs/cheerio)
> [NodeJs妹子图爬虫](http://chenxi.name/60.html)
> [通读cheerio API](https://cnodejs.org/topic/5203a71844e76d216a727d2e)

网页爬虫需要用到两个模块:`request`与`cheerio`。`request`用来请求网页，`cheerio`用来解析网页。
`cheerio`的方法与效果跟`jQuery`的方法类似
```js
var request = require('request');
var cheerio = require('cheerio');

var url = 'http://www.gamersky.com/';
request(url, function(error, response, body){
	if (!error && response.statusCode == 200) {
	  //请求网址返回的页面
	  //console.log(body);
	  //通过cheerio解析返回的网页
	  var $ = cheerio.load(body);
	  //获取新闻列表
	  var news = [];
	  //循环遍历新闻列表
	  $('li.li3').each(function(i, elem){
		//获取列表里面的div
		var $div = $('div.txt', elem);
		var link = {};
		//装载标题
		link['title'] = $('a', $div).text();
		//装载链接
		link['link'] = $('a', $div).attr('href');
		//放入到新闻列表中
		news.push(link);
	  });
	  console.log(news);
	}
});
```
