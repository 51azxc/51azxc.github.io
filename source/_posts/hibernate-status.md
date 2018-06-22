title: "Hibernate中的实体状态"
date: 2015-04-21 21:13:30
tags: ["Hibernate"]
categories: ["Java", "Hibernate"]
---

### Hibernate中的实体状态

> [写得很不错的-Hibernate中的实体状态(一)[转]](http://hi.baidu.com/kingtoon_go/item/fc0703d1e440134dfa576804)

Hibernate中实体有三个状态：瞬时、持久化和脱管。下面先来看看Hibernate对这三个状态是怎么定义的。
（1）瞬时：一个实体通过new操作符创建后，没有和Hibernate的Session建立关系，也没有手动赋值过该实体的持久化标识（持久化标识可以认为映射表的主键）。此时该实体中的任何属性的更新都不会反映到数据库表中。
（2）持久化：当一个实体和Hibernate的Session创建了关系，并获取了持久化标识，而且在Hibernate的Session生命周期内存在。此时针对该实体任何属性的更改都会直接影响到数据库表中一条记录对应字段的更新，也即与对应数据库表保持同步。
（3）脱管：当一个实体和Hibernate的Session创建了关系，并获取了持久化标识，而此时Hibernate的Session的生命周期结束，实体的持久化标识没有被改动过。针对该实体的任何属性的修改都不会及时反映到数据库表中。

这三种状态有两个很重要的点需要掌握：Hibernate的Session和持久化标识。通过这两个条件就可以判断出究竟是3种状态中的哪一个。3种不同状态通过Hibernate的Session和持久化标识可以互相之间进行转换

举个简单的例子，假如Room实体的属性id表示持久化标识，那么：
（1）创建的Room实例为瞬时状态，将表中对应的主键手动设置到id属性，此时就是脱管状态。
（2）创建的Room实例为瞬时状态，不设置id或设置的id在表中找不到对应，此时调用Hibernate Session的持久化方法，将成为持久化状态。
（3）Room实体在持久化状态下关闭Hibernate Session，此时就是脱管状态。
（4）Room实体在脱管状态下调用Hibernate Session的持久化方法，此时就是持久化状态。
（5）Room实体在持久化状态下关闭Hibernate Session，随后清除id属性的值，此时就是瞬时状态。

##### 实体状态的代码实现：瞬时—持久化
从瞬时状态到持久化状态，Hibernate提供了如下的实现，见例6.1（以下代码省略了配置映射文件的部分）。

瞬时—持久化的实现
```java
 public void run() {
      //创建瞬时状态的UserInf实例
      UserInfo userInfo = new UserInfo();
      //设置UserInfo属性，持久化标识id属性在映射中为自增长，不用设置
      userInfo.setName("RW");
      userInfo.setSex("M");
      //启动Session
      Session session = HibernateSessionFactory.currentSession();
      //启动事务
      Transaction tx = session.beginTransaction();
      //瞬时—持久化的实现，保存UserInfo代表的一条记录到数据库
      session.save(userInfo);
      //对持久化的UserInfo进行属性的更新，此时将同步数据库
      userInfo.setName("RW2");
      userInfo.setSex("F");
      //不用调用update方法，持久化状态的UserInfo会自动同步数据库
      //提交事务
      tx.commit();
      //关闭Hibernate Session
      HibernateSessionFactory.closeSession();

 }
```
针对该段代码将执行如下SQL语句：
```sql
/* session.save(userInfo);的动作 */
insert into userinfo(NAME, SEX, roomid, id)values(?, ?, ?, ?)
/* userInfo.setName("RW2"); userInfo.setSex("F"); 的动作*/
update userinfo set NAME=?,SEX=?,roomid=? where id=?
```
当瞬时状态转变为持久化状态时，需要自行调用持久化方法（如：session.save()）来执行SQL。而在持久化状态时，Hibernate控制器会自动侦测到改动，执行SQL同步数据库。

##### 实体状态的代码实现：脱管-持久化、持久化-脱管
从脱管状态和持久化状态双重转变，Hibernate提供了如下的实现
脱管状态和持久化状态双重转变
```java
public void run() {
      //创建UserInfo实例
      UserInfo userInfo = new UserInfo();
      //启动Session
      Session session = HibernateSessionFactory.currentSession();
      //启动事务
      Transaction tx = session.beginTransaction();
      //得到持久化UserInfo，此时UserInfo为持久化状态
      //与数据库中主键为11117的记录同步
      session.load(userInfo,new Long(11117));
      //提交事务
      tx.commit();
      //关闭Hibernate Session
      HibernateSessionFactory.closeSession();
      //关闭Hibernate Session后UserInfo的状态为脱管状态
      //此时依然能够得到数据库在持久化状态时的数据
      //对userInfo实体的属性的操作将不影响数据库中主键为11117的记录
      userInfo.setName("RW3");
      userInfo.setSex("M");
      //启动Session
      session = HibernateSessionFactory.currentSession();
      //启动事务
      tx = session.beginTransaction();
      //从脱管状态到持久化状态的转变，此时将更新数据库中对应主键为11117的记录
      session.update(userInfo);
      //提交事务
      tx.commit();
      //关闭Hibernate Session
      HibernateSessionFactory.closeSession();
 }
```
针对该段代码将执行如下SQL语句：
```sql
/* session.load(userInfo,new Long(11117))的动作 */
select 
        userinfo0_.id as id0_0_,
        userinfo0_.NAME as NAME0_0_,
        userinfo0_.SEX as SEX0_0_,
        userinfo0_.roomid as roomid0_0_
    from
        userinfo userinfo0_
    where
        userinfo0_.id=?
/* session.update(userInfo)的动作 */
update userinfo set NAME=?, SEX=?, roomid=? where id=?
```
可以看到`userInfo.setName("RW3")`这一部分的代码没有直接同步数据库的表，因为此时Hibernate Session已经关闭了，此时是脱管状态。而直到再次打开Hibernate Session并调用`session.update(userInfo)`，此时由于持久化标识存在于UserInfo实例，因此将从脱管状态转变为持久化状态，同步数据库。

##### 持久化方法对状态的影响
在Hibernate中定义了多个持久化方法，这些方法的调用对实体状态是有影响的。注意，并不是每一个持久化方法都会将实体状态变为持久化状态。在之前的代码中，已经使用到的持久化方法为`session.save()`、`session.load()`、`session.update()`。下面是另外一些持久化方法的调用方式。

（1）**session.delete()方法**
该方法将已经存在的表记录删除，其所影响的状态是从持久化、脱管状态转变为瞬时状态
```java
 public void run() {
      //创建UserInfo实例
      UserInfo userInfo = new UserInfo();
      //启动Session
      Session session = HibernateSessionFactory.currentSession();
      //启动事务
      Transaction tx = session.beginTransaction();
      //得到持久化UserInfo，此时UserInfo为持久化状态
      //与数据库中主键为11117的记录同步
      session.load(userInfo,new Long(11117));
      //删除持久化状态的UserInfo实体，此时UserInfo实体为瞬时状态
      session.delete(userInfo);
      //提交事务
      tx.commit();
      //关闭Hibernate Session
      HibernateSessionFactory.closeSession();
      //由于执行了session.delete因此UserInfo实体为瞬时状态，在数据库中找不到主键为11117的数据
      //此时依然能够显示该实体的属性
      System.out.println("---Id:" + userInfo.getId());
      System.out.println("---Name:" + userInfo.getName());
      System.out.println("---Sex:" + userInfo.getSex());
      //更新UserInfo实体的持久化标识，使其成为脱管状态
      userInfo.setId(11116);
      //启动Session
      session = HibernateSessionFactory.currentSession();
      //启动事务
      tx = session.beginTransaction();
      //调用delete方法将脱管状态的UserInfo实体转变为瞬时状态
      session.delete(userInfo);
      //提交事务
      tx.commit();
      //关闭Hibernate Session
      HibernateSessionFactory.closeSession();
 }
```
针对该段代码将执行如下SQL语句：
```sql
/* session.load(userInfo,new Long(11117))的动作 */
select
        userinfo0_.id as id0_0_,
        userinfo0_.NAME as NAME0_0_,
        userinfo0_.SEX as SEX0_0_,
        userinfo0_.roomid as roomid0_0_
    from
        userinfo userinfo0_
    where
        userinfo0_.id=?
/* session.delete(userInfo)的动作 */
delete
        from
            userinfo
        where
            id=?
/* session.delete(userInfo)的动作 */
delete
        from
            userinfo
        where
            id=?
```
可以看到，两句delete语句分别对应了持久化状态的UserInfo和脱管状态的UserInfo的删除动作。之后两种状态的UserInfo都会成为瞬时状态。
