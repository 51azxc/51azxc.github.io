title: spring data jpa整合
date: 2016-07-30 21:03:51
tags: ['spring data jpa']
categories: ["java", "spring"]
---

> [Spring JPA Data + Hibernate + MySQL + Maven](https://www.javacodegeeks.com/2013/05/spring-jpa-data-hibernate-mysql-maven.html)
> [Spring Data Jpa 详解 （配置篇）](http://www.cnblogs.com/liuyitian/p/4062748.html)
> [spring-data-jpa 使用](http://mybar.iteye.com/blog/1863390)
> [深入浅出学Spring Data JPA](http://sishuok.com/forum/blogPost/list/7000.html)

配置文件:
```
db.driver=com.mysql.jdbc.Driver
db.user=root
db.pwd=test
db.url=jdbc:mysql://120.25.248.154:3306/test

hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
hibernate.show_sql=true
hibernate.format_sql=true
hibernate.hbm2ddl.auto=update
```
配置类:
```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories("com.framework.dao")  //
@PropertySource("classpath:jdbc.properties")
public class DataConfig {
    //数据库连接配置
    private static final String DATABASE_DRIVER="db.driver";
	private static final String DATABASE_USERNAME="db.user";
	private static final String DATABASE_PASSWORD="db.pwd";
	private static final String DATABASE_URL="db.url";
	//Hibernate配置
	private static final String HIBERNATE_DIALECT="hibernate.dialect";
	private static final String HIBERNATE_SHOW_SQL="hibernate.show_sql";
	private static final String HIBERNATE_FORMAT_SQL="hibernate.format_sql";
	private static final String HIBERNATE_HBM2DDL_AUTO="hibernate.hbm2ddl.auto";
	//需要扫描的数据原型包
	private static final String PACKAGES_TO_SCAN="com.framework.model";
	
	@Autowired
	private Environment env;
	
	@Bean
	public DriverManagerDataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
        //读取配置文件中的信息
		dataSource.setDriverClassName(env.getRequiredProperty(DATABASE_DRIVER));
		dataSource.setUrl(env.getRequiredProperty(DATABASE_URL));
		dataSource.setUsername(env.getRequiredProperty(DATABASE_USERNAME));
		dataSource.setPassword(env.getRequiredProperty(DATABASE_PASSWORD));
		return dataSource;
	}
	
	@Bean
	public LocalContainerEntityManagerFactoryBean entityManagerFactory(){
		LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
		factory.setDataSource(dataSource());
		factory.setPackagesToScan(PACKAGES_TO_SCAN);
		factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
		factory.setJpaProperties(hibernateProperties());
		
		return factory;
	}
	
	@Bean
	public JpaTransactionManager transactionManager(){
		JpaTransactionManager manager = new JpaTransactionManager();
		manager.setEntityManagerFactory(entityManagerFactory().getObject());
		return manager;
	}
	
	private Properties hibernateProperties(){
		Properties pros = new Properties();
		pros.setProperty(HIBERNATE_DIALECT, env.getRequiredProperty(HIBERNATE_DIALECT));
		pros.setProperty(HIBERNATE_SHOW_SQL, env.getRequiredProperty(HIBERNATE_SHOW_SQL));
		pros.setProperty(HIBERNATE_FORMAT_SQL, env.getRequiredProperty(HIBERNATE_FORMAT_SQL));
		pros.setProperty(HIBERNATE_HBM2DDL_AUTO, env.getRequiredProperty(HIBERNATE_HBM2DDL_AUTO));
		
		return pros;
	}
}

@Configuration
@EnableWebMvc
@Import({DataConfig.class})   //引入上面的数据配置类
@ComponentScan(basePackages={"com.framework.controller, com.framework.service"})
public class WebMvcConfig extends WebMvcConfigurerAdapter{

    @Bean 
	public ViewResolver viewResolver(){
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/page/");
		resolver.setSuffix(".jsp");
		return resolver;
	}
	
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
	}
	
	@Override
	public void configureMessageConverters(
			List<HttpMessageConverter<?>> converters) {
		super.configureMessageConverters(converters);
		GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
		Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE).create();
		converter.setGson(gson);
		converters.add(converter);
	}
}
```
数据原型
```java
@Entity
@Table(name="user")
public class UserBean {
	private int id;
	private String username;
	private String password;
	private String createDate;
	private String timestamp;
	
	//主键，自动生成
	@Id @GeneratedValue(strategy=GenerationType.AUTO)
	public int getId() {
		return id;
	}
	@Column(unique=true,nullable=false,length=50)
	public String getUsername() {
		return username;
	}
	@Column(nullable=false,length=50)
	public String getPassword() {
		return password;
	}
	@Column(length=25)
	public String getCreateDate() {
		return createDate;
	}
	//临时状态，不会存到数据库中
	@Transient
	public String getTimestamp() {
		return timestamp;
	}
	//setter...
}
```
`DAO`层
使用了`SpringDataJpa`之后`DAO`层只要定义接口即可，无需再定义一个类实现它的方法，默认开头为`findBy`的方法都是查找数据的方法，后面还可以跟其他的关键字，比如`LessThan`,`Like`,`OrderBy`等等，与SQL关键字的功能一样。
```java
public interface UserDao extends JpaRepository<UserBean,Integer>{
    //以findBy开头的方法则是查询数据
    public UserBean findByUsernameAndPassword(String username,String password);
    //以countByd开头的方法则是统计数据
    public long countByUsername(String username);
    //@Query注解可以指定查询语句获取制定的返回数据
    @Query("select createDate from UserBean where username = :name")
	public String findByUsernameAndUsertype(@Param("name")String username);
	//nativeQuery可以使用原生SQL查询
	@Query(value="select * from user where createDate = :createDate "
			+"order by convert(uname using gbk)",nativeQuery=true)
	public List<UserBean> findByUserStatus(@Param("createDate")String createDate);
    
}
```
`JpaRepository`是继承于`PagingAndSortingRepository`,此接口提供了**分页查询**和**排序**功能，而它又继承于`CrudRepository`，此接口则提供了最基本的**增删查改**功能。接口后面跟的泛型参数为数据模型类与主键的类型。

服务层
```java
@Service
@Transactional(propagation=Propagation.REQUIRED)
public class UserService {
    @Autowired
	private UserDao userDao;
	
	public void saveUser(UserBean user) {
		userDao.save(user);
	}
	
	public void deleteUser(int id) {
		userDao.delete(id);
	}
	
	@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true)
	public boolean checkUsername(String username) {
		return userDao.countByUsername(username) > 0;
	}
	
	@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true)
	public List<UserBean> selectAll(String orderBy, String sc, int page, int size) {
		//这里的page是指第几页，而不是从多少开始，例如指定为1，则是显示 limit 1*size,size
		Pageable pageable = new PageRequest(page, size, sc.equals("asc")?Direction.ASC:Direction.DESC, orderBy);
		return userDao.findAll(pageable).getContent();
	}
	//...
}
```
在服务层，增/改操作皆为`save`，传入的类有`ID`则为修改，无则为添加

控制层
```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
	private UserService userService;
	
	@RequestMapping(value="/save", method=RequestMethod.POST)
	public void saveUser(UserBean user){
		user.setCreateDate(CommonUtil.cloneDate(Calendar.DATE, 0));
		userService.saveUser(user);
	}
	
	@RequestMapping(value="/del/{id}", method=RequestMethod.POST)
	public void userDel(@PathVariable int id){
		userService.deleteUser(id);
	}
	
	@RequestMapping(value="/list",method=RequestMethod.GET,produces="text/html;charset=UTF-8")
	public List<UserBean> list(@RequestParam String sortCol, @RequestParam String sortDir, @RequestParam int start, @RequestParam int length) {
	    List<UserBean> users = userService.selectAll(sortCol, sortDir, start, length);
	    //在配置类中配置了包装json的转换器，因此直接返回对应集合即可
	    return users;
	}
}
```
测试一下:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { TestConfig.class })
public class TestController {
	
	@Test
	public void testCheckFileInfo() throws Exception {
		mockMvc.perform(MockMvcRequestBuilders.post("/user/add")
				.param("username", "11")
				.param("password", "11"))
			.andDo(MockMvcResultHandlers.print())
			.andReturn();
	}
	
	//...
}
```
