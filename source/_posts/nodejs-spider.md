title: "NodeJs网页爬虫"
date: 2016-01-14 18:15:17
tags: "spider"
categories: "nodejs"
---

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