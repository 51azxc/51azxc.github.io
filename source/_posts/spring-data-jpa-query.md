title: spring data jpa 自定义查询
date: 2016-07-31 22:12:16
tags: ['spring data jpa']
categories: ["java", "spring"]
---

> [spring data jpa 自定义全局DAO](http://hejiantx.blog.163.com/blog/static/61867342013691040295/)
> [JPA本地查询(Native Query)的总结1](http://okcomputer2009.iteye.com/blog/397465)
> [JPA查询实体部分字段](http://xiaofan-0204.iteye.com/blog/1207958)
> [How to query data via Spring data JPA by sort and pageable both out of box？](http://stackoverflow.com/questions/10527124/how-to-query-data-via-spring-data-jpa-by-sort-and-pageable-both-out-of-box)
> [JPA : How to convert a native query result set to POJO class collection](http://stackoverflow.com/questions/13012584/jpa-how-to-convert-a-native-query-result-set-to-pojo-class-collection)

 `Spring Data Jpa`预定义的接口提供了一系列的便利方法，如果这些方法仍不能满足需求，可以自定义查询接口。
 自定义查询接口要用`@NoRepositoryBean`注解修饰,然后编写一个类实现它
```java
@NoRepositoryBean
public interface QueryDao extends JpaRepository<UserBean, Integer> {
    public List<UserBean> getUsersByCondition(UserBean bean,String orderBy, String sc, int page, int size);
    public int countUsersByCondition(UserBean bean);
}
```
接下来是接口实现类:
```java
@NoRepositoryBean
public class QueryDaoImpl implements QueryDao {
    @Autowired
	private EntityManager em;
	
	public List<UserBean> getUsersByCondition(UserBean bean,String orderBy, String sc, int page, int size) {
	String hql = "from UserBean";
		hql += getUserCriteria(bean, orderBy, sc);
		Query q = em.createQuery(hql, UserBean.class);
		setUserPara(q, bean);
		q.setFirstResult(page);
		q.setMaxResults(size);
		return q.getResultList();
	}
	
	public int countUsersByCondition(UserBean bean) {
		String hql = "select count(1) from UserBean";
		hql += getUserCriteria(bean, "", "");
		Query q = em.createQuery(hql);
		setUserPara(q, bean);
		return new Long(q.getSingleResult().toString()).intValue();
	}
	
	private String getUserCriteria(UserBean ub,String orderBy,String sc){
		String criteria = " where 1=1";
		if(!CommonUtil.nullToEmpty(ub.getUsername()).equals("")){
			criteria += " and username like :username";
		}
		if(!CommonUtil.nullToEmpty(orderBy).equals("") && !CommonUtil.nullToEmpty(sc).equals("")){
			criteria += " order by "+orderBy+" "+sc;
		}
		return criteria;
	}
	
	private void setUserPara(Query q,UserBean ub){
		if(!CommonUtil.nullToEmpty(ub.getUsername()).equals("")){
			q.setParameter("username", "%"+ub.getUsername()+"%");
		}
	}
	
	//接口自带的方法，无需全部实现，直接返回null即可
	public List<UserBean> findAll() {return null;}
	public List<UserBean> findAll(Sort sort) {return null;}
	public List<UserBean> findAll(Iterable<Integer> ids) {return null;}
}
```

默认情况下查询出来的实体都是以类的形式返回，而且所有字段都会查询出来，如果需要返回部分字段，可以使用`@SqlResultSetMappings`注解。
首选在数据模型层的`Model`类上注解`@SqlResultSetMappings`
```java
@Entity
@Table(name="user")
@SqlResultSetMappings(value={
  @SqlResultSetMapping(name="show_users",classes={
    @ConstructorResult(
	  targetClass=com.framework.bean.UserBean.class,
	  columns={
		@ColumnResult(name="id"),
		@ColumnResult(name="username"),
		@ColumnResult(name="createDate")
	})
  })
  //设置其他的映射
})
public class UserBean {
    private int id;
	private String username;
	private String password;
	private String createDate;
	private String timestamp;
	
	//必须提供对应的含有相同参数的构造函数
	public UserBean(int id, String username, String createDate) { 
	}
}
```
然后在查询中可以这样使用
```java
public long count() {
	String sql = "select count(1) from users";
	//使用原生sql查询
	Query q = em.createNativeQuery(sql);
	return new Long(q.getSingleResult().toString()).intValue();
}
public List<UserBean> getByCondition(UserBean bean,String orderBy,String sc, int start, int size){
	String sql = "select * from users";
	sql += getCriteria(bean);
	//使用原生sql查询
	Query q = em.createNativeQuery(sql,"show_users");
	setParameter(q, bean);
	//分页
	q.setFirstResult(start);
	q.setMaxResults(size);
	Pageable pageable = new PageRequest(start,size,sc.equals("asc")?Direction.ASC:Direction.DESC,orderBy);
	Page<UserBean> page = new PageImpl<UserBean>(q.getResultList(), pageable, count());
	return page.getContent();
}
```
