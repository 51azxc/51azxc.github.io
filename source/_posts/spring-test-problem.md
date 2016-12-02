title: spring test问题收集
date: 2016-07-26 15:46:12
tags: ["mvc", "test", "transaction"]
categories: ["java", "spring"]
---

### SpringTest自动回滚

> [java.lang.IllegalArgumentException: A ServletContext is required to configure default servlet handling](http://stackoverflow.com/questions/21516683/java-lang-illegalargumentexception-a-servletcontext-is-required-to-configure-de)

在**Spring Test**类配置`@Transactional`注解即可设置每个`@Test`方法自动回滚，不必手动添加`@Rollback`注解。此注解需要配置`PlatformTransactionManager`类

配置类
```java
@Configuration
@EnableWebMvc
@EnableTransactionManagement
@ComponentScan
@PropertySource(value={"classpath:config.properties"})
public class TestConfig {
	
    private static final String MYSQL_DRIVER="mysql.driver";
	private static final String MYSQL_URL="mysql.url";
	private static final String MYSQL_USERNAME="mysql.username";
	private static final String MYSQL_PASSWORD="mysql.password";

    @Autowired
	private Environment env;

    @Bean
	public DriverManagerDataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource();
        //读取配置文件中的信息
		dataSource.setDriverClassName(env.getRequiredProperty(MYSQL_DRIVER));
		dataSource.setUrl(env.getRequiredProperty(MYSQL_URL));
		dataSource.setUsername(env.getRequiredProperty(MYSQL_USERNAME));
		dataSource.setPassword(env.getRequiredProperty(MYSQL_PASSWORD));
		return dataSource;
	}
	
	@Bean
	public PlatformTransactionManager transactionManager() {
		DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource());
		return transactionManager;
	}

    @Bean
	public JdbcTemplate jdbcTemplate() {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
		return jdbcTemplate;
	}
}
```
测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { TestConfig.class })
@Transactional
public class TestController {
    @Autowired
	private WebApplicationContext wac;
	private MockMvc mockMvc;
	@Autowired
	private JdbcTemplate jdbcTemplate;

    @Before
	public void setup() {
		this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
		//每次执行测试之前插入初始化数据
		String insertItemSql = "insert into table1(id, name, create_date) " 
							 + "values (?, ?, ?)";
		jdbcTemplate.update(insertItemSql, new Object[] { 1, "a", new Date()});
	}

    @Test
	public void testGetFile() throws Exception {
		MvcResult mvcResult = mockMvc.perform(get("/files/{fileId}", "1")).andDo(print()).andExpect(status().isOk()).andReturn();
		String content = mvcResult.getResponse().getContentAsString();
		System.out.println(content);
        //执行完后自动回滚
	}
}
```
