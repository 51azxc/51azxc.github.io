title: "Redis 相关配置"
date: 2015-05-06 12:13:29
tags: "nosql"
categories: ["Database", "Redis"]
---

> [redis windows下的环境搭建](http://www.cnblogs.com/lxx/archive/2013/06/04/3116985.html)
> [可靠的Windows版Redis](http://blog.csdn.net/renfufei/article/details/41180007)
> [Windows版下载地址](https://github.com/MSOpenTech/redis/releases)

将下载的压缩包解压到任意目录下，通过命令行工具进入到相关目录下，直接输入命令
```bash
redis-server.exe redis.windows.conf
```
即可启动redis服务器，其中后面的`redis.windows.conf`为配置文件，可选输入
然后令开一个命令行进入相关目录下，输入命令
```bash
redis-cli.exe -h 127.0.0.1 -p 6379
```
指定主机名以及端口即可进入redis客户端
`redis.windows.conf`配置文件的主要配置：
```
# 包含其他的配置文件，可以使用相对路径及绝对路径
include .\path\to\local.conf

# 端口号，默认6379
port 6379

# 绑定主机地址
bind 127.0.0.1

# 当客户端闲置多长时间后关闭连接，默认为0，即禁用此功能
timeout 0

# 指定日志记录级别，四个级别：debug、verbose、notice、warning
loglevel notice

# 指定日志输出文件,stdout为输出到控制台
logfile ""

# 设置数据库的数量
database 16

# 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
# 900秒（15分钟）内有1个更改
save 900 1
# 300秒（5分钟）内有10个更改
save 300 10
# 60秒内有10000个更改
save 60 10000

# 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，
# 如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
rdbcompression yes

# 指定本地数据库文件名，默认值为dump.rdb
dbfilename dump.rdb

# 指定本地数据库存放目录
dir ./

# 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
slaveof <masterip> <masterport>

# 当master服务设置了密码保护时，slav服务连接master的密码
masterauth <master-password>

# 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
requirepass foobared

# 设置同一时间最大客户端连接数，默认无限制，
# Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。
# 当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
maxclients 10000

# 指定最大heap字节数
maxheap <bytes>

# 指定Redis最大内存限制，
# Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，
# 当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作
maxmemory <bytes>

# 指定是否在每次更新操作后进行日志记录，
# Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。
# 因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
appendonly no

# 指定更新日志条件，共有3个可选值：
# no：表示等操作系统进行数据缓存同步到磁盘（快）
# always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
# everysec：表示每秒同步一次（折衷，默认值）
appendfsync everysec

# 指定Redis内存映射文件(memory mapped file)存放的路径 默认在系统盘，会占用很大的空间，关闭redis后自动删除
heapdir <directory path(absolute or relative)>
```

----

### redis 常用命令
> [Redis常用命令](http://blog.csdn.net/ithomer/article/details/9213185)

进入redis目录在命令行输入`redis-cli`即可启动redis客户端，前提是已经打开了`redis-server`服务器，`--help`参数可以查看`redis-cli`支持的参数

* 连接操作命令
`quit`：关闭连接（connection）
`auth`：简单密码认证
`help cmd`： 查看cmd帮助，例如：help get

* 持久化
`save`：将数据同步保存到磁盘
`bgsave`：将数据异步保存到磁盘
`lastsave`：返回上次成功将数据保存到磁盘的Unix时戳
`shundown`：将数据同步保存到磁盘，然后关闭服务

* 远程服务控制
`info`：提供服务器的信息和统计
`monitor`：实时转储收到的请求
`slaveof`：改变复制策略设置
`config`：在运行时配置Redis服务器

* 清空数据库
`flushdb`: 清除当前数据库的所有keys
`flushall`: 清除所有数据库的所有keys

* 对value操作的命令
`exists key`：确认一个key是否存在
`del key`：删除一个key
`type key `：返回值的类型
`keys pattern`：返回满足给定pattern的所有key,*为所有key
`randomkey`：随机返回key空间的一个
`keyrename oldname newname`：重命名key
`dbsize`：返回当前数据库中key的数目
`expire`：设定一个key的活动时间（s）
`ttl`：获得一个key的活动时间
`select index`：按索引查询
`move key dbindex`：移动当前数据库中的key到dbindex数据库
`flushdb`：删除当前选择数据库中的所有key
`flushall`：删除所有数据库中的所有key

* 对String操作的命令
`set key value`：给数据库中名称为key的string赋予值value
`get key`：返回数据库中名称为key的string的value
`getset key value`：给名称为key的string赋予上一次的value
`mget key1 key2 … key N`：返回库中多个string的value
`setnx key value`：添加string，名称为key，值为value
`setex key time value`：向库中添加string，设定过期时间time
`mset key N value N`：批量设置多个string的值
`msetnx key N value N`：如果所有名称为key i的string都不存在
`incr key`：名称为key的string增1操作
`incrby key integer`：名称为key的string增加integer
`decr key`：名称为key的string减1操作
`decrby key integer`：名称为key的string减少integer
`append key value`：名称为key的string的值附加value
`substr key start end`：返回名称为key的string的value的子串

* 对List操作的命令
`rpush key value`：在名称为key的list尾添加一个值为value的元素
`lpush key value`：在名称为key的list头添加一个值为value的 元素
`llen key`：返回名称为key的list的长度
`lrange key start end`：返回名称为key的list中start至end之间的元素
`ltrim key start end`：截取名称为key的list
`lindex key index`：返回名称为key的list中index位置的元素
`lset key index value`：给名称为key的list中index位置的元素赋值
`lrem key count value`：删除count个key的list中值为value的元素
`lpop key`：返回并删除名称为key的list中的首元素
`rpop key`：返回并删除名称为key的list中的尾元素
`blpop key1 key2 … key N timeout`：lpop命令的block版本。**?**
`brpop key1 key2 … key N timeout`：rpop的block版本。**?**
`rpoplpush srckey dstkey`：返回并删除名称为srckey的list的尾元素，并将该元素添加到名称为dstkey的list的头部

* 对Set操作的命令
`sadd key member`：向名称为key的set中添加元素member
`srem key member`：删除名称为key的set中的元素member
`spop key`：随机返回并删除名称为key的set中一个元素
`smove srckey dstkey member`：移到集合元素
`scard key`：返回名称为key的set的基数
`sismember key member`：member是否是名称为key的set的元素
`sinter key1 key2 … key N`：求交集
`sinterstore dstkey, [keys]` ：求交集并将交集保存到dstkey的集合
`sunion key1, [keys]` ：求并集
`sunionstore dstkey, [keys]` ：求并集并将并集保存到dstkey的集合
`sdiff key1, [keys]`：求差集
`sdiffstore dstkey, [keys]`：求差集并将差集保存到dstkey的集合
`smembers key`：返回名称为key的set的所有元素
`srandmember key`：随机返回名称为key的set的一个元素

* 对Hash操作的命令
`hset key field value`：向名称为key的hash中添加元素field
`hget key field`：返回名称为key的hash中field对应的value
`hmget key [fields]`：返回名称为key的hash中field i对应的value
`hmset key [fields]`：向名称为key的hash中添加元素field 
`hincrby key field integer`：将名称为key的hash中field的value增加integer
`hexists key field`：名称为key的hash中是否存在键为field的域
`hdel key field`：删除名称为key的hash中键为field的域
`hlen key`：返回名称为key的hash中元素个数
`hkeys key`：返回名称为key的hash中所有键
`hvals key`：返回名称为key的hash中所有键对应的value
`hgetall key`：返回名称为key的hash中所有的键（field）及其对应的value

