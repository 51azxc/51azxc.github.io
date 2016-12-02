title: "jedis操作"
date: 2015-05-06 14:21:02
tags: ["nosql", "data"]
categories: ["database", "redis"]
---

### jedis 使用

> [Windows下Redis的安装使用](http://os.51cto.com/art/201403/431103.htm)
> [Jedis使用示例](http://javacrazyer.iteye.com/blog/1840161)

需要jedis包
```gradle
dependencies {
	compile(
		'redis.clients:jedis:2.7.1',
		'junit:junit:4.12' //junit包，非必需
	)
}
```

相关代码
```java
public class JedisTest {
	private JedisPool pool;
	private Jedis jedis;

	@Before
	public void setUp() throws Exception{
		pool = new JedisPool("127.0.0.1");
		jedis = pool.getResource();
	}
	
	@After
	public void finish() throws Exception{
		//清空数据
		jedis.flushDB();
	}
	
	//基本操作
	@Test
	public void testString() throws Exception{
		//存数据
		jedis.set("name", "a");
		//取数据
		System.out.println(jedis.get("name"));
		//删数据
		jedis.del("name");
		System.out.println(jedis.get("name"));
		//如果键值不存在，则存入
		jedis.setnx("name", "aa");
		System.out.println(jedis.get("name"));
		//追加数据
		jedis.append("name", "1111");
		System.out.println(jedis.get("name"));
		//设定有效期
		/*jedis.setex("name", 3, "testEx");
		System.out.println(jedis.get("name"));
		Thread.sleep(5000);
		System.out.println(jedis.get("name"));*/
		//按key,value方式存取多个map
		jedis.mset("name","abc","age","111");
		System.out.println(jedis.mget("name","age"));
		//按照key截取value的值
		System.out.println(jedis.getrange("name", 1, 2));
		//释放连接池
		//pool.returnResourceObject(jedis);
	}
	
	//操作map
	@Test
	public void testMap() throws Exception{
		Map<String,String> map = new HashMap<String, String>();
		map.put("name", "a");
		map.put("age", "11");
		jedis.hmset("student", map);
		//第一个参数为key,后面至少跟一个参数为map的key,可以跟多个key
		List<String> list = jedis.hmget("student", "name", "age");
		System.out.println(list);
		
		//删除map中的某个值
		jedis.hdel("student", "age");
		System.out.println(jedis.hmget("student", "name", "age"));
		//返回存放个数
		System.out.println(jedis.hlen("student"));
		//是否存在
		System.out.println(jedis.exists("student"));
		//返回map中的所有键值
		System.out.println(jedis.hkeys("student"));
		//返回map中的所有值
		System.out.println(jedis.hvals("student"));
		
		System.out.println(jedis.hgetAll("student"));
	}
	//操作列表list
	@Test
	public void testList() throws Exception{
		//开始之前先删除所有相关数据
		jedis.del("students");
		//新数据放在左边
		jedis.lpush("students", "1");
		jedis.lpush("students", "2");
		jedis.lpush("students", "3");
		//排序,只对数字有效
		SortingParams sp = new SortingParams();
		sp.desc();
		sp.limit(0, 1);
		System.out.println(jedis.sort("students", sp));
		//新数据放在右边
		jedis.rpush("students", "s4");
		//按key取出列表数据，最后一个参数-1表示所有数据
		System.out.println(jedis.lrange("students", 0, -1));
		//获取数组长度
		System.out.println(jedis.llen("students"));
		//截取指定区间的数据
		System.out.println(jedis.lrange("students", 0, 1));
		//指定修改
		jedis.lset("students", 1, "update");
		//获取指定下标的值
		System.out.println(jedis.lindex("students", 1));
		//删除指定下标的值
		System.out.println(jedis.lrem("students", 1, "update"));
		//删除指定区间以外的值
		System.out.println(jedis.ltrim("students", 0, 1));
		//列表出栈
		System.out.println(jedis.lpop("students"));
		System.out.println(jedis.lpop("students"));
		System.out.println(jedis.lpop("students"));
	}
	//操作set
	@Test
	public void testSet() throws Exception{
		//存入数据
		jedis.sadd("ss", "aa");
		jedis.sadd("ss", "bb");
		jedis.sadd("ss", "cc");
		jedis.sadd("ss", "dd");
		jedis.sadd("ss", "ee");
		//移除数据
		jedis.srem("ss", "ee");
		//遍历
		System.out.println(jedis.smembers("ss"));
		//判断元素是否存在于set中
		System.out.println(jedis.sismember("ss", "ee"));
		//返回集合中第一个元素?
		System.out.println(jedis.srandmember("ss"));
		//返回元素个数
		System.out.println(jedis.scard("ss"));
		//出栈
		System.out.println(jedis.spop("ss"));
		System.out.println(jedis.smembers("ss"));
		
		jedis.sadd("st", "aa");
		jedis.sadd("st", "bbb");
		jedis.sadd("st", "dd");
		jedis.sadd("st", "ee");
		//交集
		System.out.println(jedis.sinter("ss","st"));
		//并集
		System.out.println(jedis.sunion("ss","st"));
		//差集
		System.out.println(jedis.sdiff("ss","st"));
		
		jedis.zadd("zs", 30, "a1");
		jedis.zadd("zs", 10, "a2");
		jedis.zadd("zs", 20, "a3");
		//根据中间的参数排序集合
		System.out.println(jedis.zrange("zs", 0, -1));
		System.out.println(jedis.zrevrange("zs", 0, -1));
	}
}
```
