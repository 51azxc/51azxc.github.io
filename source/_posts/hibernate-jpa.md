title: "hibernate注解"
date: 2015-05-13 13:52:06
tags: ["orm", "annotation"]
categories: ["java", "hibernate"]
---

### 注解标注的位置

> 来自 [hibernate注解中JPA@标注的位置](http://2694306.blog.51cto.com/2684306/1323707)

对于JPA注解，最好放在方法前，而不要放在属性名前(Best Practice)，因为属性一般都是私有的，放属性前面会破坏java的封装性，一般不要直接访问私有成员变量。此外最好还要保持field和get，set方法的一致。
如果写在属性前面，所有字段都要写在属性前；如果写在getter方法前面，所有字段都写在getter方法前面；必须保持一致

通过看官方文档了解到注解写在属性和方法上面的区别：

使用hibernate注解时，可以选择对类的属性或方法进行注解，根据注解位置不同，hibernate的访问类型分别为field或property。

EJB3规范要求在需要访问的元素上进行注解声明，例如：如果访问类型为 property就要在getter方法上进行注解声明,，如果访问类型为 field就要在字段上进行注解声明。

应该尽量避免混合使用这两种访问类型。（混用这两者造成程序报错，如value too long，但是很难找到问题所在） Hibernate根据@Id 或 @EmbeddedId的位置来判断访问类型。

若访问类型被标以"property"，则Hibernate会扫描getter方法的注解,若访问类型被标以"field"，则扫描字段的注解.否则,扫描标为`@Id`或`@embeddedId`的元素。

你可以覆盖某个属性(property)的访问类型，但是受注解的元素将不受影响： 例如一个具有field访问类型的实体,(我们)可以将某个字段标注为 `@AccessType("property")`, 则该字段的访问类型随之将成为property,但是其他字段上依然需要携带注解.

----------

### Hibernate JPA注解说明

> 来自 
> [Hibernate JPA注解说明](http://www.cnblogs.com/shudonghe/archive/2013/01/18/2866484.html)
> [Hibernate注解三个常见问题](http://www.blogjava.net/allrounder/articles/323591.html)

`@Entity(name="")`: 必须,name为可选,对应数据库中一的个表
`@Table(name="",catalog="",schema="")`: 可选,通常和@Entity配合使用,只能标注在实体的class定义处,表示实体对应的数据库表的信息

* `name`:可选,表示表的名称.默认地,表名和实体名称一致,只有在不一致的情况下才需要指定表名
* `catalog`:可选,表示Catalog名称,默认为`Catalog("")`.
* `schema`:可选,表示Schema名称,默认为`Schema("")`.

`@id`: 必须,`@id`定义了映射到数据库表的主键的属性,一个实体只能有一个属性被映射为主键.置于`getXxxx()`前.
`@GeneratedValue(strategy=GenerationType,generator="")`: 可选

* `strategy`:表示主键生成策略,有`AUTO`,`INDENTITY`,`SEQUENCE` 和 `TABLE` 4种,分别表示让ORM框架自动选择,根据数据库的Identity字段生成,根据数据库表的Sequence字段生成,以有根据一个额外的表生成主键,默认为`AUTO`
* `generator`:表示主键生成器的名称,这个属性通常和ORM框架相关,例如,Hibernate可以指定uuid等主键生成方式.

`@Basic(fetch=FetchType,optional=true)`: 可选,表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的getXxxx()方法,默认即为`@Basic`

* `fetch`: 表示该属性的读取策略,有`EAGER`和`LAZY`两种,分别表示主支紧急和延迟加载,默认为`EAGER`.
* `optional`:表示该属性是否允许为null,默认为`true`

`@Column`: 可选,描述了数据库表中该字段的详细定义

* `name`: 表示数据库表中该字段的名称,默认情形属性名称一致
* `nullable`: 表示该字段是否允许为null,默认为true
* `unique`: 表示该字段是否是唯一标识,默认为false
* `length`: 表示该字段的大小,仅对String类型的字段有效
* `insertable`: 表示在ORM框架执行插入操作时,该字段是否应出现INSETRT语句中,默认为`true`
* `updateable`: 表示在ORM框架执行更新操作时,该字段是否应该出现在UPDATE语句中,默认为`true`.对于一经创建就不可以更改的字段,该属性非常有用,如对于birthday字段.
* `columnDefinition`: 表示该字段在数据库中的实际类型.通常ORM框架可以根据属性类型自动判断数据库中字段的类型,但是对于Date类型仍无法确定数据库中字段类型究竟是DATE,TIME还是TIMESTAMP.此外,String的默认映射类型为VARCHAR,如果要将String类型映射到特定数据库的BLOB或TEXT字段类型.

`@Transient`: 可选,表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性.如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic
`@ManyToOne(fetch=FetchType,cascade=CascadeType)`: 可选,表示一个多对一的映射,该注解标注的属性通常是数据库表的外键

* `optional`: 是否允许该字段为null,该属性应该根据数据库表的外键约束来确定,默认为`true`
* `fetch`: 表示抓取策略,默认为`FetchType.EAGER`
* `cascade`: 表示默认的级联操作策略,可以指定为`ALL`,`PERSIST`,`MERGE`,`REFRESH`和`REMOVE`中的若干组合,默认为无级联操作
* `targetEntity`: 表示该属性关联的实体类型.该属性通常不必指定,ORM框架根据属性类型自动判断targetEntity.
示例：
```java
//订单Order和用户User是一个ManyToOne的关系
//在Order类中定义
@ManyToOne()
@JoinColumn(name="USER")
public User getUser() {
    return user;
}
```

`@JoinColumn`: 可选,描述的不是一个简单字段,而是一个关联字段,例如.描述一个@ManyToOne的字段

* `name`: 该字段的名称.由于`@JoinColumn`描述的是一个关联字段,如ManyToOne,则默认的名称由其关联的实体决定.

`@OneToMany(fetch=FetchType,cascade=CascadeType)`: 可选,描述一个一对多的关联,该属性应该为集体类型,在数据库中并没有实际字段.

* `fetch`: 表示抓取策略,默认为`FetchType.LAZY`,因为关联的多个对象通常不必从数据库预先读取到内存
* `cascade`: 表示级联操作策略,对于OneToMany类型的关联非常重要,通常该实体更新或删除时,其关联的实体也应当被更新或删除
示例：
```java
@OneTyMany(cascade=ALL)
public List getOrders() {
   return orders;
}
```

`@OneToOne(fetch=FetchType,cascade=CascadeType)`: 可选,描述一个一对一的关联

* `fetch`: 表示抓取策略,默认为`FetchType.LAZY`
* `cascade`: 表示级联操作策略

`@ManyToMany`: 可选,描述一个多对多的关联.多对多关联上是两个一对多关联,但是在ManyToMany描述中,中间表是由ORM框架自动处理

* `targetEntity`: 表示多对多关联的另一个实体类的全名,例如:package.Book.class
* `mappedBy`: 表示多对多关联的另一个实体类的对应集合属性名称
示例:
User实体表示用户,Book实体表示书籍,为了描述用户收藏的书籍,可以在User和Book之间建立ManyToMany关联
```java
@Entity
public class User {
   private List books;
   @ManyToMany(targetEntity=package.Book.class)
   public List getBooks() {
       return books;
   }
   public void setBooks(List books) {
       this.books=books;
   }
}
@Entity
public class Book {
   private List users;
   @ManyToMany(targetEntity=package.Users.class, mappedBy="books")
   public List getUsers() {
       return users;
   }
   public void setUsers(List users) {
       this.users=users;
   }
}
```
两个实体间相互关联的属性必须标记为`@ManyToMany`,并相互指定targetEntity属性,
需要注意的是,有且只有一个实体的`@ManyToMany`注解需要指定`mappedBy`属性,指向targetEntity的集合属性名称
利用ORM工具自动生成的表除了User和Book表外,还自动生成了一个User_Book表,用于实现多对多关联

`@MappedSuperclass`: 可选,可以将超类的JPA注解传递给子类,使子类能够继承超类的JPA注解
示例:
```java
@MappedSuperclass
public class Employee() {}

@Entity
public class Engineer extends Employee {}

@Entity
public class Manager extends Employee {}
```

`@Embedded`: 可选,`@Embedded`将几个字段组合成一个类,并作为整个Entity的一个属性.
例如User包括id,name,city,street,zip属性.
我们希望city,street,zip属性映射为Address对象.这样,User对象将具有id,name和address这三个属性.
Address对象必须定义为`@Embededable`
示例:
```java
@Embeddable
public class Address {city,street,zip}
@Entity
public class User {
    @Embedded
    public Address getAddress() {}
}
```

----------

### MySQL中Text, MeduimText, LongText在Hibernate中的设置

> 来自 [MySQL中Text, MeduimText, LongText在Hibernate中的设置](http://blog.csdn.net/eagelangel/article/details/8534632)

```java
@Lob(type = LobType.CLOB, fetch = FetchType.LAZY)
//Hibernate会对应到MySQL的MeduimText上去。MedumnText最大16777215字节
//如果length = 16777215则对应到LongText。LongText最大2147483647字节
@Column(length = 16777215)
public String getXXX(){
    return xxx;
}
```
`@Lob` 通常与`@Basic`同时使用，提高访问速度。
```java
@Lob 
@Basic(fetch = FetchType.LAZY) 
@Column(name="DtaContent", columnDefinition="CLOB", nullable=true) 
public String getDtaContent() {
    return dtaContent;
}
```
如果提示为found text,excepted clob
改为`columnDefinition="TEXT"`就可以了。

----------

### hibernate lazy fetch
> 来自 
> [hibernate lazy fetch](http://wanghaoran04141205.iteye.com/blog/519881)
> [Hibernate检索策略之5.1类级别检索策略——Hibernate4究竟怎么玩](http://www.cnblogs.com/tingyuxuan007/archive/2012/08/28/2661022.html)

lazy是延迟加载，默认是延迟加载。
主要是为了系统的性能，当一张表引用到另外一张表时，如果不是立即需要另外一张表的内容，就可以采取延迟加载，直到要用到时才加载另外一张表。
`fetch` 和 `lazy` 主要是用来级联查询的，
而 `cascade` 和 `inverse` 主要是用来级联插入和修改的      
fetch参数指定了关联对象抓取的方式是select查询还是join查询，
select方式时先查询返回要查询的主体对象（列表），再根据关联外键 id，
每一个对象发一个select查询，获取关联的对象，形成n+1次查 询；
而join方式，主体对象和关联对象用一句外键关联的sql同时查询出来，不会形成多次查询。
如果你的关联对象是延迟加载的，它当然不会去查询关联对象。 另外，在hql查询中配置文件中设置的join方式是不起作用的（而在所有其他查询方式如get、criteria或再关联获取等等都是有效的），会使用 select方式，除非你在hql中指定join fetch某个关联对象。fetch策略用于定义 get/load一个对象时，如何获取非lazy的对象/集合。 这些参数在Query中无效。

----------

#### Hibernate的cascade属性

> 来自
> [Hibernate的cascade属性 特别是 cascadeType.all的 作用](http://blog.csdn.net/z69183787/article/details/22327725)
> [Hibernate基础之十：一对多关联的CRUD__@ManyToOne(cascade=(CascadeType.ALL))](http://blog.csdn.net/null____/article/details/8159497)

JPA中的`CascadeType.ALL`并不等于`{CascadeType.PESIST,CascadeType.REMOVE,CascadeType.MERGE,CascadeType.REFRESH}`
在Hibernate中调用`session.save()` or `session.update()`并不能触发 `{CascadeType.PESIST,CascadeType.REMOVE,CascadeType.MERGE,CascadeType.REFRESH}` 的级联操作，而能触发`CascadeType.ALL`的级联。如不希望用`CascadeType.ALL`，需要使用Hibernate自身对 cascade的注解
```
@Cascade(value=org.hibernate.annotations.CascadeType.SAVE_UPDATE)
```

----------

假定一个组里有n多用户，但是一个用户只对应一个用户组。

1.所以Group对于Users是“一对多”的关联关系`@OneToMany`,Users对于Group是“多对一”`@ManyToOne`
2.CRUD时候，希望是能从具体用户Users查到其对应的Group，反过来也能通过Group查到具体Users，所以是双向关联（所以要用mappedBy去除冗余信息）
```java
@Entity  
@Table(name="t_Group")//指定一个表名  
public class Group   
{  
    private int id;  
    private String name;  
    private Set<Users> users = new HashSet<Users>();  
  
    @Id  
    @GeneratedValue//主键用自增序列  
    public int getId() {  
        return id;  
    }  
    @OneToMany(mappedBy="group",cascade=(CascadeType.ALL))//以“多”一方为主导管理，级联用ALL  
    public Set<Users> getUsers() {  
        return users;  
    } 
    //setter
}

@Entity  
@Table(name="t_Users")  
public class Users   
{  
    private int id;  
    private String name;  
    private Group group;  
  
    @Id  
    @GeneratedValue  
    public int getId() {  
        return id;  
    }  
    @ManyToOne(fetch=FetchType.LAZY,cascade=(CascadeType.ALL))//解决1+N,级联用ALL 
    @JoinColumn(name="groupId")//指定外键名称，不指定的默认值是group_Id  
    public Group getGroup() {  
        return group;  
    }
    //setter
|
```

**C增**
cascade：级联，只影响cud，不影响r（all全都级联，persist存储时级联，remove删除时级联）
如果没有设置cascade，默认需要save（Group)和save(users)，两个都要存，设置级联之后，只存一个就行了
级联依赖于这句：`@ManyToOne(cascade=(CascadeType.ALL))`//需要依赖于其他的东西时候
设置好正反向之后，多个有级联关系的对象就一起被保存了
```java
Users u1 = new Users();  
Users u2 = new Users();  
u1.setName("u1");  
u2.setName("u2");  
//u1和u2的id自增  
 
Group g = new Group();  
g.setName("g1");  
//g的id自增  
 
g.getUsers().add(u1);//正向  
g.getUsers().add(u2);  
 
u1.setGroup(g);//反向  
u2.setGroup(g);//不然u1和u2中的group信息为空  
 
session.save(g);//因为设置级联，所以存储g时候也把u1和u2存上了。  
//不设置级联的话，还要存储u1和u2
```

**R查**
默认会这样处理（平时管用的思路也是这样）：
1.取“多”的时候，把“一”取出来
2.取“一”时，不取“多”的，用到时候再去取（看user信息时候一般看组名，看group时候user信息太多不必看）
fetch管读取，cascade管增删改
`@OneToMany(mappedBy="group",cascade=(CascadeType.ALL),fetch=FetchType.EAGER)`
`@OneToMany`默认的是`LAZY`，`@ManyToOne`默认是`EAGER`
```java
Users u = (Users)session.get(Users.class,11);//取id为11号的u  
//hibernate产生的语句里也把group拿出来了：group1_.id as id0_0_,和group1_.name as name0_0_ 

Hibernate:   
    select  
        users0_.id as id1_1_,  
        users0_.groupId as groupId1_1_,  
        users0_.name as name1_1_,  
        group1_.id as id0_0_,  
        group1_.name as name0_0_   
    from  
        t_Users users0_   
    left outer join  
        t_Group group1_   
            on users0_.groupId=group1_.id   
    where  
        users0_.id=?  

//只取出Group的话，不会去查询里边的user  
Group group = (Group)session.get(Group.class,11); 

Hibernate:   
    select  
        group0_.id as id0_0_,  
        group0_.name as name0_0_   
    from  
        t_Group group0_   
    where  
        group0_.id=? 
```

**U更新**
注意：fetch影响两者读取顺序（两边都设成EAGER要多取出一次）
`@OneToMany`,`@ManyToOne`都写`cascade=(CascadeType.ALL)`
update时候自动关联更新
```java
//因为cascade=(CascadeType.ALL)，所以自动关联更新  
Users u = (Users)session.load(Users.class,11);//取id为11号的u  
u.setName("u250");  
u.getGroup().setName("gp01");
```

**D删**
删多：实测只删掉目的项目，不关联其他
先load（就是select）一下，确认有之后，再删
```java
Users u1 = new Users();  
u1.setId(18);  
u1.setGroup(null);//严谨起见，应该先让俩表脱离关联  
session.delete(u1);

//hql删除
s.createQuery("delete from User u where u.id = 1").executeUpdate();//User是类名
```
