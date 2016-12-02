title: Mongoose拾遗
date: 2016-04-24 00:34:09
tags: ["nosql", "mongodb"]
categories: ["nodejs", "mongoose"]
---

### 加密字段

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

### 关联ID

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

### 转换时区

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
