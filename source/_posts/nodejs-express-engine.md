title: Express中替换模板为HTML
date: 2016-04-23 22:46:15
tags: "express"
categories: ["nodejs", "express"]
---

> [Why is express telling me that my default view engine is not defined?](http://stackoverflow.com/questions/17560760/why-is-express-telling-me-that-my-default-view-engine-is-not-defined)

**Express4**中的默认模板为*Jade*，如果要替换为*html*则需要引入*ejs*模板再指定模板文件：
```js
var ejs = require('ejs');
app.set('view engine', 'html');
app.engine('html', ejs.renderFile);
```
