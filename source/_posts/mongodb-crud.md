title: "MongoDB简单使用"
date: 2015-05-07 16:51:03
tags: "nosql"
categories: ["Database", "MongoDB"]
---

### MongoDB增删查改

> [8天学通MongoDB——第二天 细说增删查改](http://www.cnblogs.com/huangxincheng/archive/2012/02/19/2357846.html)
> [mongodb_修改器（`$inc/$set/$unset/$push/$pop/upsert`）](http://blog.csdn.net/mcpang/article/details/7752736)
> [MongoDB基本命令用](http://www.cnblogs.com/xusir/archive/2012/12/24/2830957.html)

#### `insert`操作
```js
//创建一个集合
db.createCollection("students");
var student = {"sno":1,"sname":"s1",age:12,"course":["chinese","math"]};
db.students.insert(student);

student.sno=2;
student.sname="s2";
student.age=14;
student.course=["math","english"];

db.students.insert(student);

//查询所有记录
db.students.find();
```
这里已经成功的写入了2条记录

如果需要批量插入，可以使用`for`循环语句来插入
```js
//删除所有记录
db.students.remove("");

for(var i=1; i<=10; i++){
  var student = {"sno":i,"sname":"s"+i,age:i,"course":["chinese","math"]};
  db.students.insert(student);
}

db.students.find();
```
`save()`函数可以添加数据，添加的列随意指定，如果调用了集合中默认的主键`_id`，则会进行更新操作
```js
//插入新数据
db.students.save({sno:123,sname:"ss",age:15});
//更新数据
var student = db.students.findOne();
student.age = 11;
db.students.save(student);
```

----------

#### `select`操作
操作符对应 `>`为`$gt`,`>=`为`$gte`,`<`为`$lt`,`<=`为`$lte`,`!=`为`$ne`
关系连接符对应 `or`为`$or`, `in`为`$in`, `not in`为`$nin`

```js
//默认每页显示20条记录，如果需要下一页使用it命令
db.studens.find();
select * from students

db.students.find({"age":1,"sname":"s1"});
select * from students where age = 1 and sname = "s1"

db.students.find({"sno":{$gt:5}, "course": {$in:["math"] } });
select * from students where sno > 5 and course in ("math")

db.students.find({$or: [{"sname":"s2"},{"age":5}]});
select * from students where sname = "s2" or age = 5

//查询指定属性
db.students.find({},{sname:1});
select sname from students
//sname也可以用true或false,当用ture的情况下和sname:1效果一样，
//如果用false就是排除sname，显示sname以外的列信息

//通过正则表达式实现模糊查询
db.students.find({"sname":/6/},{age:1})
select age from students where sname like '%6%'
db.students.find({"sname":/^s/,"sname":/5$/});
select * from students where sname like 's%' and sname like '%s'

//使用where方法来实现条件查询
db.students.find({$where:function(){return this.sno==7 || this.age==8 }});
select * from students where sno = 7 or age = 8

//排序,1为升序，-1为降序
db.students.find().sort({sno:-1});

//limit用于查询限定数之前的记录，skip用于查询限定数之后的记录
//两者组合可以用于分页，limit是pageSize,skip为页数*pageSize
db.students.find().limit(10).skip(5);

//查询第一条记录，可以跟条件查询
db.students.findOne();

//统计行数
db.students.count();
select count(*) from students
db.students.find({sno: {$exists: true}}).count();
select count(sno) from students
db.students.count({age:1});
select count age from students where age = 1;

//过滤重复数据
db.students.distinct("age");
select distinct age from students;
```

----------

#### `group`操作
`group`主要的参数:
`key`: 需要分组的属性
`initial`: 每组都分享一个”初始化函数“，特别注意：是每一组，比如这个的age=20的value的list分享一个**?**
`$reduce`: 这个函数的第一个参数是当前的文档对象，第二个参数是上一次function操作的累计对象，第一次为`initial`中的{"students"：[]}。有多少个文档， `$reduce`就会调用多少次`initial`函数，age=22同样也分享一个`initial`函数
`condition`: 过滤条件
`finalize`: 这是个函数，每一组文档执行完后，多会触发此方法，那么在每组集合里面加上count也就是它的活了
```js
db.students.group({
  "key": {"age":true},
  "initial": {"students":[]},
  "$reduce": function(cur,prev){
    prev.students.push(cur.sname);
  },
  "condition": {"age":{$gte:10}},
  "finalize": function(out){
    out.count = out.students.length;
  }
});
```
执行的部分结果为：
```json
[
        {
                "age" : 11,
                "students" : [
                        "s1",
                        "s2"
                ],
                "count" : 2
        },
        {
                "age" : 12,
                "students" : [
                        "s5",
                        "s3",
                        "s4"
                ],
                "count" : 3
        },
        {
                "age" : 14,
                "students" : [
                        "s9",
                        "s10"
                ],
                "count" : 2
        },
        {
                "age" : 15,
                "students" : [
                        "ss"
                ],
                "count" : 1
        }
]
```

----------

#### `Map-Reduce`操作

[官网示例](http://docs.mongodb.org/manual/core/map-reduce/)

`mapReduce`其实是一种编程模型，用在分布式计算中，其中有一个`map`函数，这个称为映射函数，里面会调用`emit(key,value)`，集合会按照你指定的key进行映射分组。
`reduce`函数为简化函数，会对map分组后的数据进行分组简化，注意：在`reduce(key,value)`中的key就是emit中的key，vlaue为emit分组后的emit(value)的集合，这里也就是很多{"count":1}的数组
`mapReduce`则为最后的执行函数
```js
db.students.mapReduce(
  //map
  function(){
   emit(this.age, {count:1});
  },
  //reduce
  function(key, value){
    var result = {count: 0};
    for(var i=0; i<value.length; i++){
      result.count += value[i].count;
    }
    return result;
  },
  {
    "query": {"sname":/^s/},
    "out": "collection"
  }
);

db.collection.find();
```
执行结果为
```json
{
        "result" : "collection",    //存放的集合名
        "timeMillis" : 514,
        "counts" : {
                "input" : 11,       //传入文档的个数
                "emit" : 11,        //此函数被调用的次数
                "reduce" : 3,       //此函数被调用的次数?
                "output" : 7        //最后返回文档的个数
        },
        "ok" : 1
}
> db.collection.find();
{ "_id" : 6, "value" : { "count" : 1 } }
{ "_id" : 7, "value" : { "count" : 1 } }
{ "_id" : 8, "value" : { "count" : 1 } }
{ "_id" : 11, "value" : { "count" : 2 } }
{ "_id" : 12, "value" : { "count" : 3 } }
{ "_id" : 14, "value" : { "count" : 2 } }
{ "_id" : 15, "value" : { "count" : 1 } }
```

----------

#### 游标查询
```js
var cursor = db.students.find();
//循环输出完毕，游标自动销毁，再次输入cursor则为空
while(cursor.hasNext()){
  //输出json格式
  printjson(cursor.next());
}

//使用forEach进行迭代
var cursor = db.students.find().sort({"age":-1}).limit(10).skip(5);
cursor.forEach(function(x){
  print(x.sname);
});

//通过数组形式来访问游标数据
var cursor = db.students.find();
cursor[3];

//将游标转换成数组
var arr = db.students.find().toArray();
print(arr.length);
```

----------

#### 索引
```js
//创建索引
db.students.ensureIndex({sname:1,age:-1});

//唯一索引
db.students.ensureIndex({sname:1,{"unique":true}});

//查看当前集合的索引
db.students.getIndexes();

//查看所有索引记录大小
db.students.totalIndexSize();

//读取当前集合的所有index信息
db.students.reIndex();

//删除指定索引
db.students.dropIndex("sname_1_age_-1");

//删除所有索引
db.students.dropIndexes();
```

----------

#### `update`操作
```js
//$set即可对指定属性进行修改
db.students.update({sno:2},{$set:{age:11}});
update students set age=11 where sno=2;
//如果属性不存在则会创建该属性
db.students.update({sno:2},{$set:{teacher:"t2"}});

//$unset用来删除指定属性
db.students.update({sno:2},{$unset:{teacher:1}});

//$inc可以对数字型属性进行加法操作
db.students.update({sno:2},{$inc: {age: -9}});
update students set age=age-9 where sno=2;

//$push向文档的某个数组类型的键添加一个数组元素，
//不过滤重复的数据。添加时键存在，要求键值类型必须是数组；
//键不存在，则创建数组类型的键
db.students.update({sno:2},{$push: {course:"english"}});

//$addToSet可以避免插入重复的数据
db.students.update({sno:3},{$addToSet: {course:"english"}});
db.students.update({sno:2},{$addToSet: {course:"english"}});

//$pop类似栈的pop操作，可以从数组头部(-1),尾部(1|0)来取出元素，即从数组中删除元素
db.students.update({sno:2},{$pop: {course:1}});

//$pull从数组中删除指定元素
db.students.update({sno:3},{$pull:{course:"math"}});

//使用$或者数组下标可以定位到数组元素进行修改
db.students.update({sno:2},{$set:{"course.0":"chinese"}});

//upsert是insert or update的结合体，有则更新，无则新增，
//只要指定update的第三个参数为true即可
//update还有第四个参数,如果指定true,如果匹配多条则全部更新，默认只更新第一条
db.students.update({sno:4},{$set:{teacher:"t5"}},true);
```

----------

#### 其他操作
`explain`函数可以用来性能分析
```js
> db.students.find().explain();
{
        "cursor" : "BasicCursor",
        "isMultiKey" : false,
        "n" : 11,
        "nscannedObjects" : 11,
        "nscanned" : 11,
        "nscannedObjectsAllPlans" : 11,
        "nscannedAllPlans" : 11,
        "scanAndOrder" : false,
        "indexOnly" : false,
        "nYields" : 0,
        "nChunkSkips" : 0,
        "millis" : 0,
        "server" : "Ezio-Hu:27017",
        "filterSet" : false
}
```
部分参数说明
`cursor`: `BasicCursor`这里的查找采用的是“表扫描”，也就是顺序查找
`nscanned`: 数据库浏览的文档数目
`n`: 最终返回的文档数
`millis`: 查询耗时

`tojson`: 将一个对象转换成json格式的对象
```js
tojson(new Object('student'))
```

`printjson`: 将获取的对象输出json格式


