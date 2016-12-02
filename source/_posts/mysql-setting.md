title: "MySQL基本配置"
date: 2015-05-05 11:27:30
tags: ["sql", "backup"]
categories: ["database", "mysql"]
---

### MySQL免安装版配置

> [MySQL Win免安装版配置](http://blog.csdn.net/zhouqi_2011/article/details/8751636)
> [MySQL-5.6.13免安装版配置方法](http://blog.csdn.net/q98842674/article/details/12094777)
> [安装 mysql-5.7.5-m15-winx64](http://www.cnblogs.com/wenthink/p/MySQLInstall.html)

修改文件目录下的`my-default.ini`为`my.ini`
```ini
[mysqld]

basedir = D:/mysql
datadir = D:/mysql/data
port = 3306
server_id = 1
log-error = "D:/mysql/log/mysql_error_log.err"
# 服务端使用的字符集
character-set-server = utf8
# mysql服务器支持的最大并发连接数（用户数）
max_connections = 100
# 设置table高速缓存的数量
table_open_cache = 256
# 查询缓存大小，用于缓存SELECT查询结果
query_cache_size = 1M
# 内存中的每个临时表允许的最大大小
tmp_table_size = 32M
# 缓存的最大线程数
thread_cache_size = 8

# InnoDB
innodb_data_home_dir = D:/mysql/data
# 事务相关参数
# 如果值为1,则InnoDB在每次commit都会将事务日志写入磁盘（磁盘IO消耗较大），这样保证了完全的ACID特性。
# 如果值为0,则表示事务日志写入内存log和内存log写入磁盘的频率都为1次/秒。
# 如果值为2,则表示事务日志在每次commit都写入内存log，但内存log写入磁盘的频率为1次/秒。
innodb_flush_log_at_trx_commit = 1
# InnoDB日志数据缓冲大小
innodb_log_buffer_size = 2M
# InnoDB使用缓冲池来缓存索引和行数据。该值设置的越大，则磁盘IO越少。
innodb_buffer_pool_size = 128M
# 每一个InnoDB事务日志的大小
innodb_log_file_size = 32M
# InnoDB内核最大并发线程数
innodb_thread_concurrency = 8

join_buffer_size = 128M
sort_buffer_size = 2M
read_rnd_buffer_size = 2M
explicit_defaults_for_timestamp = true
sql-mode = STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```
改后保存，进入命令行工具，定位到mysql目录下的bin目录里，在命令行输入
```bash
mysqld --install MySQL --defaults-file="D:/mysql/my.ini"
```
即可注册MySQL服务为系统服务，然后通过`net stary MySQL`启动，或者使用`service.msc`命令启动服务管理界面启动MySQL服务

* 5.7版本之后首先输入命令`mysqld --initialize-insecure`初始化

----

### 密码设置

> [Mysql 免安装版 root@localhost 密码设置](http://blog.csdn.net/renxianzuo/article/details/6026765)

使用`set password`命令
```bash
mysql -u root
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```
使用mysqladmin
```bash
mysqladmin -u root password "newpass"
```
如果root已经设置过密码，采用如下方法
```bash
mysqladmin -u root password oldpass "newpass"
```
编辑user表
```bash
mysql -u root
use mysql;
UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
FLUSH PRIVILEGES;
```
5.7之后，上述语句无效，应为
```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```
终极办法
```bash
mysql -u root mysql
UPDATE user SET password=PASSWORD("new password") WHERE user='root';
FLUSH PRIVILEGES;
```

----

### 数据备份
#### 全量备份

> [windows mysql 自动备份的几种方法](http://blog.csdn.net/younkerjqb/article/details/12193245)

```bash
rem *******************************Code Start*****************************
@echo off
set "Ymd=%date:~,4%%date:~5,2%%date:~8,2%"
C:\MySQL\bin\mysqldump --opt -u root --password=123456 bbs > D:\db_backup\bbs_%Ymd%.sql

@echo on
rem *******************************Code End*****************************

```

将以上脚本代码保存为bat文件
然后使用Windows的“计划任务”定时执行该脚本即可
注意要备份的路径必须存在
通过`%date:~5,2%`来组合得出当前日期，组合的效果为**yyyymmdd**,date命令得到的日期格式默认为**yyyy-mm-dd**(如果不是此格式可以通过pause命令来暂停命令行窗口看通过`%date:~,20%`得到的当前计算机日期格式)，所以通`过%date:~5,2%`即可得到日期中的第五个字符开始的两个字符，例如今天为2015-05-04,通过`%date:~5,2%`则可以得到05。（日期的字符串的下标是从0开始的）

----

#### 增量备份

> [【SQL】MySQL之使用mysqlbinlog进行增量备份及恢复详解](http://blog.csdn.net/jueblog/article/details/9909669)

在`my.ini`中添加如下语句
```ini
log-bin="C:/Program Files/mysql-5.6.22-winx64/logbin/log"
expire_logs_days=7
```
`log-bin`为记录日志的文件路径，最后的`/log`为文件名
`expire_log_days`为指定间隔多少日后删除所有的日志文件
重启mysql服务后，在指定文件夹logbin下可以发现有log.index,log.000001这样的文件。其中log.index为备份文件的索引，指明有哪些备份文件,其他的为备份文件，存放用户对数据库的所有操作
log.index的文件内容如下
```ini
C:\Program Files\mysql-5.6.22-winx64\logbin\log.000001
C:\Program Files\mysql-5.6.22-winx64\logbin\log.000002
C:\Program Files\mysql-5.6.22-winx64\logbin\log.000003
```
通过`mysqlbinlog`程序可以看到日志文件的内容
```bash
mysqlbinlog "C:\Program Files\mysql-5.6.22-winx64\logbin\log.000001"
```
按时间导出其中的内容
```bash
mysqlbinlog --start-datetime="2015-05-04 00:00:00" --stop-datetime="2015-05-04 23:59:59" juelog.000001 -r test.sql
```
这样可以把处于时间段的所有操作记录导入到test.sql文件中，test.sql文件在mysqlbinlog同级目录。
`--start-datetime`与`--stop-datetime`是可选参数

**按位置进行恢复**
清空表后输入
```bash
mysqlbinlog --stop-position="行数" log.000001 | mysql -u root -p
```
输入密码后即可恢复数据

**总结**
Mysql数据库会以二进制形式，自动把用户对mysql数据库的操作，记录到备份文件中。
当用户希望恢复的时候，可以使用备份文件，来进行相应的恢复。
备份文件中会记录创建表的语句、删除表的语句、insert语句、delect语句、update语句等，而不会记录select语句。
增量备份记录的内容包括：
1. 操作语句本身。
2. 操作的时间。
3. 操作的位置。

----

### MySQL 5.6 中 TIMESTAMP 的变化

> [MySQL 5.6 中 TIMESTAMP 的变化](http://www.williamsang.com/archives/818.html)

在my.ini中的`[mysqld]`节点下添加
```ini
explicit_defaults_for_timestamp=true
```
重启MySQL后错误消失，这时**TIMESTAMP**的行为如下：
* **TIMESTAMP**如果没有显示声明**NOT NULL**，是允许**NULL**值的，可以直接设置改列为**NULL**，而没有默认填充行为。
* **TIMESTAMP**不会默认分配`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`属性。
* 声明为**NOT NULL**且没有默认子句的TIMESTAMP列是没有默认值的。往数据表中插入列，又没有给**TIMESTAMP**列赋值时，如果是严格SQL模式，会抛出一个错误，如果严格SQL模式没有启用，该列会赋值为'0000-00-00 00:00:00′，同时出现一个警告。（这和MySQL处理其他时间类型数据一样，如DATETIME）

----

### CommunicationsException异常处理
> 来自 [com.mysql.jdbc.exceptions.jdbc4.CommunicationsException](http://flex4.blog.163.com/blog/static/2116401192012113104932833/)

mysql5将其连接的等待时间(wait_timeout)缺省为8小时.如果在wait_timeout秒期间内，数据库连接(java.sql.Connection)一直处于等待状态，mysql5就将该连接关闭。这时，你的JavaEE应用的连接池仍然合法地持有该连接的引用。当用该连接来进行数据库操作时，就碰到上述错误。
解决这个错误需要在my.ini配置文件中`[mysqld]`节点下配置`wait_timeout`将超时时间改为自己需要的时间
```ini
[mysqld]
wait_timeout=1814400 
```

----

### Mac下的MySQL卸载

> [How do you uninstall MySQL from Mac OS X?](http://stackoverflow.com/questions/1436425/how-do-you-uninstall-mysql-from-mac-os-x)
> [Remove MySQL completely from Mac OSX](https://gist.github.com/vitorbritto/0555879fe4414d18569d)

运行终端，输入下列命令
```bash
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/MySQL*
rm -rf ~/Library/PreferencePanes/MySQL*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*
sudo rm -rf /private/var/db/receipts/*mysql*
```