title: "MongoDB配置及部分命令"
date: 2015-05-06 17:47:35
tags: "nosql"
categories: ["Database", "MongoDB"]
---

### MongoDB安装

> [第一节 MongoDB介绍及下载与安装](http://www.cnblogs.com/mecity/archive/2011/06/11/2078527.html)

至官网下载了压缩包后解压至任意目录，然后在目录下建立data文件夹，并在data文件夹下建立db与log两个文件夹，db文件夹用来存放数据存储文件,log文件夹用来存放日志文件。在log文件夹下建立默认的日志危机，如MongoDB.log。
这里采用注册为系统服务的方式安装。在命令行cmd中cd到mongodb目录下，运行
```bash
mongod --dbpath "C:\mongodb\data\db" --logpath "C:\mongodb\data\log\MongoDB.log" --install --serviceName "MongoDB"
```
命令即可注册名为`MongoDB`的服务，如果成功，可以通过`net start MongoDB`启动服务。
点击bin目录下的mongo.exe打开数据库可以测试一下有无成功

还可以直接通过相关程序来运行，在命令行中输入以下命令
```bash
mongod -dbpath "C:\mongodb\data\db"
```
执行此命令即将mongodb的数据库文件创建到`C:\mongodb\data\db`目录,关闭的华可以直接双机mongod.exe

--fork 以守护进程方式运行MongoDB，创建服务器进程
```bash
mongod --port 10220 --fork  --dbpath "C:\mongodb\data\db" --logpath "C:\mongodb\data\log\MongoDB.log"
```

如果需要停止mongodb,直接关闭的话可能会造成数据丢失，稳妥的方式为
```bash
user admin
db.shutdownServer();
```

----------

### MongoDB相关命令

> [MongoDB基本命令用](http://www.cnblogs.com/xusir/archive/2012/12/24/2830957.html)

`help`: 显示帮助命令
`show dbs`: 显示所有数据库
`show collections`: 显示当前数据库所有集合(类似表)
`show users`: 显示所有用户
`show logs`: 显示所有日志名
`show log [name]`: 根据日志名输出日志到控制台

`use <db_name>`: 切换数据库
`db.foo.find()`: 查询集合foo的所有数据
`db.cloneDatabase("127.0.0.1")`: 将指定机器上的数据库的数据克隆到当前数据库
`db.copyDatabase("mydb", "temp", "127.0.0.1")`: 将本机的mydb的数据复制到temp数据库中
`db.repairDatabase()`: 修复当前数据库
`db.getName()`: 查看当前使用的数据库
`db.stats()`: 查看当前数据库状态
`db.version()`: 查看当前数据库版本
`db.getMongo()`: 查看当前db的主机地址

`db.auth("username","password")`: 数据库安全认证
`db.addUser("username")`: 添加新用户
`db.addUser("username", "password", true)`: 最后一个参数为只读选项
`db.removeUser("username")`: 删除用户

`db.createCollection("collName", {size: 20, capped: 5, max: 100})`: 创建一个聚集集合
`db.getCollection("collName")`: 得到指定名称的聚集集合
`db.getCollectionNames()`: 得到当前db的所有聚集集合
`db.printCollectionStats()`: 显示当前db所有聚集索引的状态
`db.foo.renameCollection("newCollName")`: 集合重命名
`db.foo.dataSize()`: 查看数据空间大小
`db.foo.stats()`: 查看集合状态
`db.userInfo.totalSize()`: 查看集合总大小
`db.userInfo.storageSize()`: 查看集合存储空间大小

`db.getPrevError()`: 查询之前的错误信息
`db.resetError()`: 清除错误记录
