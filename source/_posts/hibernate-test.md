title: "hibernate 实例"
date: 2015-05-13 17:49:45
tags: "orm"
categories: ["java", "hibernate"]
---

### hibernate 实例

> [Hibernate4实战 之 第二部分：Hibernate的基本配置](http://sishuok.com/forum/blogPost/list/2465.html)
> [Hibernate4.3 创建SessionFactory](http://bbs.csdn.net/topics/390452637)
> [hibernate4.0中SessionFactory的创建](http://blog.csdn.net/zl3450341/article/details/8640005)
> [Hibernate4 Annotation实例](http://blog.csdn.net/xwin1989/article/details/7380736)
> [Hibernate4使用Annotation连接访问MySQL的小例子](http://blog.csdn.net/quchj89/article/details/7563078)
> [Hibernate 4.3.0.Final get session](http://stackoverflow.com/questions/20870369/hibernate-4-3-0-final-get-session)

gradle.build配置文件
```gradle
apply plugin: 'java'
apply plugin: 'eclipse'

repositories {
	maven { 
	  url "http://maven.oschina.net/content/groups/public/"
	  url "http://maven.oschina.net/content/repositories/thirdparty/"
	}
	mavenCentral()
}

dependencies {
	compile(
		'org.hibernate:hibernate-core:4.3.6.Final',
		'org.hibernate:hibernate-c3p0:4.3.6.Final',
		'mysql:mysql-connector-java:5.1.6',
		'junit:junit:4.12'
	)
}
```
hibernate.cfg.xml配置文件，位于src目录下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN" "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd"> 
<hibernate-configuration>
	<session-factory>
	    <property name="hibernate.connection.provider_class">org.hibernate.c3p0.internal.C3P0ConnectionProvider</property>
		<property name="hibernate.c3p0.min_size">2</property>
		<property name="hibernate.c3p0.max_size">20</property>
		<property name="hibernate.c3p0.timeout">120</property>
		<property name="hibernate.c3p0.max_statements">10</property>
		<property name="hibernate.c3p0.idle_test_period">300</property>
		<property name="hibernate.c3p0.acquire_increment">2</property>
        
        <property name="connection.url">jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8</property>
        <property name="connection.username">root</property>
        <property name="connection.password">root</property>
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
	    
        <property name="show_sql">true</property>  
        <property name="format_sql">true</property>
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>  
        <property name="hibernate.current_session_context_class">thread</property>  
        <property name="hbm2ddl.auto">update</property>  
	</session-factory>    
</hibernate-configuration>
```
HibernateUtil类
```java
public class HibernateUtil {

	private static SessionFactory sessionFactory = buildSessionFactory();
	
	private static SessionFactory buildSessionFactory(){
		try{
			// 读取Hibernate的配置文件  hibernate.cfg.xml文件 
			Configuration cfg = new Configuration().configure();
			//添加实体类
			cfg.addAnnotatedClass(Student.class);
			ServiceRegistry sr = new StandardServiceRegistryBuilder().applySettings(cfg.getProperties()).build();
			return cfg.buildSessionFactory(sr);
		}catch(Exception e){
			e.printStackTrace();
		}
		return null;
	}
	
	public static SessionFactory getSessionFactory(){
		return sessionFactory;
	}
	
}
```
实体类
```java
@Entity
@Table(name="student")
public class Student {

	private int sid;
	private String sname;
	private int age;
	
	@Id @GeneratedValue(strategy=GenerationType.AUTO)
	public int getSid() {
		return sid;
	}
	@Column(name="sname",length=50)
	public String getSname() {
		return sname;
	}
	public int getAge() {
		return age;
	}
	public void setSid(int sid) {
		this.sid = sid;
	}
	public void setSname(String sname) {
		this.sname = sname;
	}
	public void setAge(int age) {
		this.age = age;
	}
}
```
测试类
```java
public class Test1 {
	private Session session;
	
	@Before
	public void before(){
		//使用getCurrentSession方法对于crud操作必须添加事务，openSession需手动添加事物及关闭session
		session = HibernateUtil.getSessionFactory().getCurrentSession();
	}
	
	@Test
	public void testInsert(){
		Student s = new Student();
		s.setSname("b");
		s.setAge(14);
		session.beginTransaction();
		session.save(s);
		session.getTransaction().commit();
	}
	
	@Test
	public void testUpdate1(){
		session.beginTransaction();
		Student s = (Student) session.get(Student.class, 1);
		if(s!=null){
			System.out.println(s.getAge());
			s.setAge(12);
		}
		session.getTransaction().commit();
	}
	
	@Test
	public void testUpdate2(){
		String hql = "update Student s set s.age = :age where s.sname=:name";
		session.beginTransaction();
		Query q = session.createQuery(hql);
		q.setParameter("age", 15);
		q.setParameter("name", "a");
		int i = q.executeUpdate();
		System.out.println(i);
		session.getTransaction().commit();
	}
	
	@Test
	public void testSelect(){
		String hql = "from Student s where s.sname=:name";
		Transaction t = session.beginTransaction();
		Query q = session.createQuery(hql);
		q.setParameter("name", "b");
		Student s = (Student) q.uniqueResult();
		System.out.println(s.getSname()+" "+s.getAge());
		t.commit();
	}
	
	@Test
	public void testDelete(){
		Transaction t = session.beginTransaction();
		Student s = (Student) session.get(Student.class, 1);
		session.delete(s);
		t.commit();
	}
	
}
```
