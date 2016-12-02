title: "Hibernate 查询"
date: 2015-04-21 21:48:31
tags: ["orm", "hql", "sql", "criteria"]
categories: ["java", "hibernate"]
---

### HQL语句

> [hibernate各种条件查询汇总（对象、单字段、多字段等）](http://hi.baidu.com/tnzzjh/item/5a10973626a9d7c02f8ec23e)
> [Hibernate HQL查询 插入 更新实例](http://blog.csdn.net/shimiso/article/details/7241712)
> [Hql语句注意事项总结（转载）](http://beckrabbit.iteye.com/blog/133152)
> [Hibernate的HQL中in参数设置](http://blog.csdn.net/liyanhui1001/article/details/7301764)
> [Hibernate -- 使用Hibernate获取最大值(max)的三种方法(转)](http://blog.csdn.net/aierda/article/details/6218672)
> [Hibernate中的query.setFirstResult(),query.setMaxResults();](http://blog.csdn.net/switzerland/article/details/3127992)

#### 查询所有字段
```java
String hql = "from Users";      
Query query = session.createQuery(hql);      
List<Users> users = query.list();      
for(Users user : users){      
 System.out.println(user.getName() + " : " + user.getPasswd() + " : " + user.getId());     
}
```

#### 查询单个字段
```java
String hql = " select name from Users";      
Query query = session.createQuery(hql);
List<String> list = query.list();      
for(String str : list){      
   System.out.println(str);      
}
```

#### 查询其中几个字段
```java
String hql = " select name,passwd from Users";      
Query query = session.createQuery(hql);      
//默认查询出来的list里存放的是一个Object数组      
List<Object[]> list = query.list();      
for(Object[] object : list){      
    String name = (String)object[0];      
    String passwd = (String)object[1];      
 System.out.println(name + " : " + passwd);      
}
```

#### 默认查询结果以List形式返回
```java
//查询其中几个字段,添加new list(),注意list里的l是小写的。也不需要导入包，这样通过query.list()出来的list里存放的不再是默认的Object数组了，而是List集合了  
String hql = " select new list(name,passwd) from Users";   
Query query = session.createQuery(hql);   
//默认查询出来的list里存放的是一个Object数组，但是在这里list里存放的不再是默认的Object数组了，而是List集合了   
List<List> list = query.list();   
for(List user : list){   
 String name = (String)user.get(0);   
 String passwd = (String)user.get(1);   
 System.out.println(name + " : " + passwd);   
}
```

#### 默认查询结果以Map形式返回 
```java
//查询其中几个字段,添加new map(),注意map里的m是小写的。也不需要导入包，这样通过query.list()出来的list里存放的不再是默认的Object数组了，而是map集合了     
String hql = " select new map(name,passwd) from Users";      
Query query = session.createQuery(hql);      
//默认查询出来的list里存放的是一个Object数组，但是在这里list里存放的不再是默认的Object数组了，而是Map集合了     
List<Map> list = query.list();      
for(Map user : list){      
    //一条记录里所有的字段值都是map里的一个元素,key是字符串0,1,2,3....，value是字段值      
    //如果将hql改为：String hql = " select new map(name as username,passwd as password) from Users";,那么key将不是字符串0,1,2...了，而是"username","password"了     
 String name = (String)user.get("0");//get("0");是get(key),注意:0,1,2...是字符串，而不是整形     
 String passwd = (String)user.get("1");      
 System.out.println(name + " : " + passwd);      
} 
```

修改默认查询结果(query.list())不以Object[]数组形式返回，以Set形式返回,但是因为Set里是不允许有重复的元素，所以:username和password的值不能相同。只需将hql改为：`String hql = " select new set(name,passwd) from Users"`;

#### 默认查询结果以自定义类返回
```java
//必须加包名，且自定义类必须要有相应的带参构造函数
String hql = " select new  com.test.model.UserBean(name,passwd) from Users";
Query query = session.createQuery(hql); 
List<UserBean> users = query.list();
```

#### 条件查询
**索引方式**
```java
String hql = "from Users where name=? and password=?"; 
Query query = session.createQuery(hql);
query.setString(0, "name");
query.setString(1, "password");
```
**自定义参数名**
```java
String hql = "from Users where name=:username and password=:password";     
Query query = session.createQuery(hql);
query.setParameter("username", "name",Hibernate.STRING);
//对于某些参数类型 setParameter()方法可以更具参数值的Java类型，
//猜测出对应的映射类型，因此这时不需要显示写出映射类型
//但是对于一些类型就必须写明映射类型，比如 java.util.Date类型，
//因为它会对应Hibernate的多种映射类型，比如Hibernate.DATA或者 Hibernate.TIMESTAMP
query.setParameter("password", "password"); 
```
**数组条件查询**
```java
String hql = "from table1 a where a.field1 in (:alist)";
Query query = session.createQuery(hql);
//a可以是数组,也可以是List
query.setParameterList("alist",a);
```

#### 获取最大值
```java
//hql一定要加上别名
String hql = "select max(a.id) from table1 a";
int max = ((Long)session.createQuery(hql).uniqueResult()).intValue();

//sql需要给查询字段加上别名
String sql = "select max(id) as maxid from table1"
int max = ((Long)session.createSQLQuery(sql).addScalar("maxId", Hibernate.INTEGER).uniqueResult()).intValue();

//criteria
int max = ((Long)session.createCriteria(table1.class).setProjection( Projections.projectionList().add(Projections.max("id " ) ) ).uniqueResult()).intValue();
```

----------

### Criteria查询
> [Hibernate中Criteria的完整用法](http://www.cnblogs.com/JemBai/archive/2010/01/06/1640636.html)

_QBC (Query By Criteria)_ 主要有`Criteria`,`Criterion`,`Oder`,`Restrictions`类组成

| HQL运算符 | QBC运算符 | 含义 |
| --- | --- | --- |
| `=` | Restrictions.eq() | 等于 |
| `<>` | Restrictions.not(Exprission.eq()) | 不等于 |
| `>` | Restrictions.gt() | 大于 |
| `>=` | Restrictions.ge() | 大于等于 |
| `<` | Restrictions.lt() | 小于 |
| `<=` | Restrictions.le() | 小于等于 |
| is null | Restrictions.isnull() | 等于空值 |
| is not null | Restrictions.isNotNull() | 非空值 |
| like | Restrictions.like() | 字符串模式匹配 |
| and | Restrictions.and()/conjunction() | 逻辑与 |
| or | Restrictions.or()/disjunction() | 逻辑或 |
| not | Restrictions.not() | 逻辑非 |
| in(列表) | Restrictions.in() | 等于列表中的某一个值 |
| not in(列表) | Restrictions.not(Restrictions.in()) | 不等于列表中任意一个值 |
| between x and y | Restrictions.between() | 闭区间xy中的任意值 |
| not between x and y | Restrictions.not(Restrictions..between()) | 小于值X或者大于值y |

#### 创建一个Criteria 实例
org.hibernate.Criteria接口表示特定持久类的一个查询。Session是 Criteria实例的工厂。
```java
Criteria c = session.createCriteria(UserBean.class);
List list = c.list();
```

#### 限制结果集内容
```java
Criteria c = session.createCriteria(UserBean.class).add( Restrictions.like("name", "a%") );
List list = c.list();
//约束可以按逻辑分组
List list = session.createCriteria(UserBean.class)
    .add( Restrictions.in( "name", new String[] { "a", "b", "c" } ) )
    .add( Restrictions.or(
        Restrictions.eq( "age", new Integer(0) ),
        Restrictions.isNull("age")
) ).list();
```
Hibernate提供了相当多的内置criterion类型(Restrictions 子类), 但是尤其有用的是可以允许你直接使用SQL
```java
//{alias}占位符应当被替换为被查询实体的列别名
Criteria c = session.createCriteria(UserBean.class).add( Restrictions.sql("lower({alias}.name) like lower(?)", "a%", Hibernate.STRING) );
List list = c.list();
```
Property实例是获得一个条件的另外一种途径。你可以通过调用`Property.forName()` 创建一个Property。
```java
Property age = Property.forName("age");
List list = session.createCriteria(UserBean.class)
    .add( Property.forName("name").in( new String[] { "a", "b", "c" } ) )
    .add( Restrictions.disjunction().add(age.isNull() )
) ).list();
```

#### 结果集排序
使用org.hibernate.criterion.Order来为查询结果排序
```java
Criteria c = session.createCriteria(UserBean.class)
    .addOrder(Order.asc("age") ).setMaxResults(10);
List list = c.list();
```

#### 关联
使用`createCriteria()`非常容易的在互相关联的实体间建立约束
```java
List cats = sess.createCriteria(Cat.class)
    .add( Restrictions.like("name", "F%")
    .createCriteria("kittens")
    .add( Restrictions.like("name", "F%")
.list();
```
注意第二个`createCriteria()`返回一个新的 Criteria实例，该实例引用kittens 集合中的元素。

接下来，替换形态在某些情况下也是很有用的。
```java
//createAlias()并不创建一个新的 Criteria实例
List cats = sess.createCriteria(Cat.class)
    .createAlias("kittens", "kt")
    .createAlias("mate", "mt")
    .add( Restrictions.eqProperty("kt.name", "mt.name") )
.list();
```
Cat实例所保存的之前两次查询所返回的kittens集合是 没有被条件预过滤的。如果你希望只获得符合条件的kittens， 你必须使用`returnMaps()`。
```java
List cats = sess.createCriteria(Cat.class)
    .createCriteria("kittens", "kt")
    .add( Restrictions.eq("name", "F%") )
.returnMaps()
.list();

Iterator iter = cats.iterator();
while ( iter.hasNext() ) {
    Map map = (Map) iter.next();
    Cat cat = (Cat) map.get(Criteria.ROOT_ALIAS);
    Cat kitten = (Cat) map.get("kt");
}
```

#### Example查询示例
org.hibernate.criterion.Example类允许你通过一个给定实例 构建一个条件查询。
```java
Cat cat = new Cat();
cat.setSex('F');
cat.setColor(Color.BLACK);
List results = session.createCriteria(Cat.class).add( Example.create(cat) )
.list();

//使用examples在关联对象上放置条件
List results = session.createCriteria(Cat.class)
    .add( Example.create(cat) )
    .createCriteria("mate")
    .add( Example.create( cat.getMate() ) )
.list();
```

#### 投影(Projections)、聚合（aggregation）和分组（grouping）
org.hibernate.criterion.Projections是 Projection 的实例工厂。我们通过调用 setProjection()应用投影到一个查询
```java
List results = session.createCriteria(Cat.class)
    .setProjection( Projections.rowCount() )
    .add( Restrictions.eq("color", Color.BLACK) )
.list();

List results = session.createCriteria(Cat.class)
    .setProjection( Projections.projectionList()
    .add( Projections.rowCount() )
    .add( Projections.avg("weight") )
    .add( Projections.max("weight") )
    .add( Projections.groupProperty("color") ))
.list();
```
在一个条件查询中没有必要显式的使用 "group by" 。某些投影类型就是被定义为 分组投影，他们也出现在SQL的group by子句中。
你可以选择把一个别名指派给一个投影，这样可以使投影值被约束或排序所引用。下面是两种不同的实现方式：
```java
List results = session.createCriteria(Cat.class)
    .setProjection( Projections.alias(Projections.groupProperty("color"), "colr" ) )
    .addOrder( Order.asc("colr") )
.list();

List results = session.createCriteria(Cat.class)
    .setProjection( Projections.groupProperty("color").as("colr") )
    .addOrder( Order.asc("colr") )
.list();
```
`alias()`和`as()`方法简便的将一个投影实例包装到另外一个 别名的Projection实例中。简而言之，当你添加一个投影到一个投影列表中时 你可以为它指定一个别名：
```
List results = session.createCriteria(Cat.class)
    .setProjection( Projections.projectionList()
    .add( Projections.rowCount(), "catCountByColor" )
    .add( Projections.avg("weight"), "avgWeight" )
    .add( Projections.max("weight"), "maxWeight" )
    .add( Projections.groupProperty("color"), "color" )
).addOrder( Order.desc("catCountByColor") )
 .addOrder( Order.desc("avgWeight") )
.list();

List results = session.createCriteria(Domestic.class, "cat")
    .createAlias("kittens", "kit")
    .setProjection( Projections.projectionList()
    .add( Projections.property("cat.name"), "catName" )
    .add( Projections.property("kit.name"), "kitName" )
).addOrder( Order.asc("catName") )
 .addOrder( Order.asc("kitName") )
.list();
```
你也可以使用`Property.forName()`来表示投影：
```java
List results = session.createCriteria(Cat.class)
    .setProjection( Property.forName("name") )
    .add( Property.forName("color").eq(Color.BLACK) )
.list();

List results = session.createCriteria(Cat.class)
    .setProjection( Projections.projectionList()
    .add( Projections.rowCount().as("catCountByColor") )
    .add( Property.forName("weight").avg().as("avgWeight") )
    .add( Property.forName("weight").max().as("maxWeight") )
    .add( Property.forName("color").group().as("color" )
).addOrder( Order.desc("catCountByColor") )
 .addOrder( Order.desc("avgWeight") )
.list();
```

#### 离线(detached)查询和子查询
DetachedCriteria类使你在一个session范围之外创建一个查询，并且可以使用任意的 Session来执行它
```java
DetachedCriteria query = DetachedCriteria.forClass(Cat.class)
    .add( Property.forName("sex").eq('F') );

Transaction txn = session.beginTransaction();
List results = query.getExecutableCriteria(session).setMaxResults(100).list();
txn.commit();
session.close();
```
DetachedCriteria也可以用以表示子查询。条件实例包含子查询可以通过 Subqueries或者Property获得
```java
DetachedCriteria avgWeight = DetachedCriteria.forClass(Cat.class).setProjection( Property.forName("weight").avg() );
    
session.createCriteria(Cat.class).add( Property.forName("weight).gt(avgWeight) ).list();

DetachedCriteria weights = DetachedCriteria.forClass(Cat.class)
    .setProjection( Property.forName("weight") );

session.createCriteria(Cat.class).add( Subqueries.geAll("weight", weights) ).list();
```
甚至相互关联的子查询也是有可能的：
```java
DetachedCriteria avgWeightForSex = DetachedCriteria.forClass(Cat.class, "cat2")
    .setProjection( Property.forName("weight").avg() )
    .add( Property.forName("cat2.sex").eqProperty("cat.sex") );

session.createCriteria(Cat.class, "cat").add( Property.forName("weight).gt(avgWeightForSex) ).list();
```

----------

### SQL语句
> [使用SQLQuery 在Hibernate中使用sql语句](http://www.cnblogs.com/biGpython/archive/2012/03/26/2417926.html)

#### 1.标量查询（Scalar queries）
最基本的SQL查询就是获得一个标量（数值）的列表。
```java
sess.createSQLQuery("SELECT * FROM CATS").list();
sess.createSQLQuery("SELECT ID, NAME, BIRTHDATE FROM CATS").list();
```
它们都将返回一个Object数组(Object[])组成的List，数组每个元素都是CATS表的一个字段值。Hibernate会使用ResultSetMetadata来判定返回的标量值的实际顺序和类型。
如果要避免过多的使用**ResultSetMetadata**,或者只是为了更加明确的指名返回值，可以使用`addScalar()`。
```java
sess.createSQLQuery("SELECT * FROM CATS").addScalar("ID", Hibernate.LONG).addScalar("NAME", Hibernate.STRING).addScalar("BIRTHDATE", Hibernate.DATE)
```
这个查询指定了:

* SQL查询字符串
* 要返回的字段和类型

它仍然会返回Object数组,但是此时不再使用ResultSetMetdata,而是明确的将ID,NAME和BIRTHDATE按照Long,String和Short类型从resultset中取出。同时，也指明了就算query是使用*来查询的，可能获得超过列出的这三个字段，也仅仅会返回这三个字段。

对全部或者部分的标量值不设置类型信息也是可以的。
```java
sess.createSQLQuery("SELECT * FROM CATS") .addScalar("ID", Hibernate.LONG) .addScalar("NAME") .addScalar("BIRTHDATE")
```
基本上这和前面一个查询相同,只是此时使用`ResultSetMetaData`来决定NAME和BIRTHDATE的类型，而ID的类型是明确指出的。

关于从`ResultSetMetaData`返回的java.sql.Types是如何映射到Hibernate类型，是由**方言(Dialect)**控制的。假若某个指定的类型没有被映射，或者不是你所预期的类型，你可以通过Dialet的registerHibernateType调用自行定义。

#### 2.实体查询(Entity queries)
上面的查询都是返回标量值的，也就是从resultset中返回的“裸”数据。下面展示如何通过`addEntity()`让原生查询返回实体对象。
```java
sess.createSQLQuery("SELECT * FROM CATS").addEntity(Cat.class); 
sess.createSQLQuery("SELECT ID, NAME, BIRTHDATE FROM CATS").addEntity(Cat.class);
```
这个查询指定：

* SQL查询字符串
* 要返回的实体

假设Cat被映射为拥有ID,NAME和BIRTHDATE三个字段的类，以上的两个查询都返回一个List，每个元素都是一个Cat实体。

假若实体在映射时有一个`many-to-one`的关联指向另外一个实体，在查询时必须也返回那个实体，否则会导致发生一个`column not found`的数据库错误。这些附加的字段可以使用`*`标注来自动返回，但我们希望还是明确指明，看下面这个具有指向Dog的`many-to-one`的例子：
```java
sess.createSQLQuery("SELECT ID, NAME, BIRTHDATE, DOG_ID FROM CATS").addEntity(Cat.class);
```
这样`cat.getDog()`就能正常运作

#### 处理关联和集合类(Handling associations and collections)
通过提前抓取将Dog连接获得，而避免初始化proxy带来的额外开销也是可能的。这是通过`addJoin()`方法进行的，这个方法可以让你将关联或集合连接进来。
```java
sess.createSQLQuery("SELECT c.ID, NAME, BIRTHDATE, DOG_ID, D_ID, D_NAME FROM CATS c, DOGS d WHERE c.DOG_ID = d.D_ID")
    .addEntity("cat", Cat.class)
    .addJoin("cat.dog");
```
上面这个例子中，返回的Cat对象，其dog属性被完全初始化了，不再需要数据库的额外操作。注意，我们加了一个别名("cat")，以便指明join的目标属性路径。通过同样的提前连接也可以作用于集合类，例如，假若Cat有一个指向Dog的一对多关联。
```java
sess.createSQLQuery("SELECT ID, NAME, BIRTHDATE, D_ID, D_NAME, CAT_ID FROM CATS c, DOGS d WHERE c.ID = d.CAT_ID")
    .addEntity("cat", Cat.class)
    .addJoin("cat.dogs");
```
到此为止，我们碰到了天花板：若不对SQL查询进行增强，这些已经是在Hibernate中使用原生SQL查询所能做到的最大可能了。
下面的问题即将出现：返回多个同样类型的实体怎么办？或者默认的别名/字段不够又怎么办？

#### 返回多个实体(Returning multiple entities)
到目前为止,结果集字段名被假定为和映射文件中指定的的字段名是一致的。假若SQL查询连接了多个表，同一个字段名可能在多个表中出现多次，这就会造成问题。

下面的查询中需要使用字段别名注射（这个例子本身会失败）：
```java
sess.createSQLQuery("SELECT c.*, m.* FROM CATS c, CATS m WHERE c.MOTHER_ID = c.ID")
    .addEntity("cat", Cat.class)
    .addEntity("mother", Cat.class)
```
这个查询的本意是希望每行返回两个Cat实例，一个是cat,另一个是它的妈妈。但是因为它们的字段名被映射为相同的，而且在某些数据库中，返回的字段别名是“c.ID”,"c.NAME"这样的形式，而它们和在映射文件中的名字（"ID"和"NAME"）不匹配，这就会造成失败。

下面的形式可以解决字段名重复：
```java
sess.createSQLQuery("SELECT {cat.*}, {mother.*} FROM CATS c, CATS m WHERE c.MOTHER_ID = c.ID")
    .addEntity("cat", Cat.class)
    .addEntity("mother", Cat.class)
```
这个查询指明：

* SQL查询语句，其中包含占位附来让Hibernate注射字段别名
* 查询返回的实体

上面使用的`{cat.*}`和`{mother.*}`标记是作为“所有属性”的简写形式出现的。当然你也可以明确地罗列出字段名，但在这个例子里面我们让Hibernate来为每个属性注射SQL字段别名。字段别名的占位符是属性名加上表别名的前缀。在下面的例子中，我们从另外一个表（cat_log）中通过映射元数据中的指定获取Cat和它的妈妈。注意，要是我们愿意，我们甚至可以在where子句中使用属性别名。
```java
String sql = "SELECT ID as {c.id}, NAME as {c.name}, " 
    + "BIRTHDATE as {c.birthDate}, MOTHER_ID as {c.mother}, {mother.*} " 
    + "FROM CAT_LOG c, CAT_LOG m WHERE {c.mother} = c.ID"; 
List loggedCats = sess.createSQLQuery(sql)
    .addEntity("cat", Cat.class)
    .addEntity("mother", Cat.class)
    .list()
```

#### 返回非受管实体(Returning non-managed entities)
可以对原生sql 查询使用ResultTransformer。这会返回不受Hibernate管理的实体。
```java
sess.createSQLQuery("SELECT NAME, BIRTHDATE FROM CATS")
    .setResultTransformer(Transformers.aliasToBean(CatDTO.class))
```
这个查询指定：

* SQL查询字符串
* 结果转换器(result transformer)

上面的查询将会返回CatDTO的列表,它将被实例化并且将NAME和BIRTHDAY的值注射入对应的属性或者字段。

#### 参数（Parameters）
原生查询支持位置参数和命名参数：
```java
Query query = sess.createSQLQuery("SELECT * FROM CATS WHERE NAME like ?").addEntity(Cat.class); 
List pusList = query.setString(0, "Pus%").list(); 
query = sess.createSQLQuery("SELECT * FROM CATS WHERE NAME like :name").addEntity(Cat.class); 
List pusList = query.setString("name", "Pus%").list();
```
