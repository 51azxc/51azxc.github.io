title: spring mvc 文件上传下载
date: 2016-07-26 14:58:22
tags: ["mvc", "file", "test"]
categories: ["java", "spring"]
---

> [Spring: Make your java-based configuration more elegant](https://www.javacodegeeks.com/2013/11/spring-make-your-java-based-configuration-more-elegant.html)
> [Simple Spring MVC Web Application using Gradle](http://www.tuicool.com/articles/3UzMzi)
> [SpringMVC中的文件上传](http://blog.csdn.net/jadyer/article/details/7575934)
> [Spring MVC 4 File Download Example](http://websystique.com/springmvc/spring-mvc-4-file-download-example/)
> [Spring MVC File Download Example](http://keylesson.com/index.php/2015/06/25/spring-mvc-file-download-example-2119/)
> [Using Spring MVC Test to unit test multipart POST request](http://stackoverflow.com/questions/21800726/using-spring-mvc-test-to-unit-test-multipart-post-request)

### 文件上传

配置类
```java
@Configuration
@EnableWebMvc
@ComponentScan
@PropertySources({
	@PropertySource(value={"file:${configPath}/config.properties"}, ignoreResourceNotFound=true),
	@PropertySource("classpath:config.properties")
})
public class WebMvcConfig extends WebMvcConfigurerAdapter{
	@Bean 
	public ViewResolver viewResolver(){
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/page/");
		resolver.setSuffix(".jsp");
		return resolver;
	}
	
	@Bean
	public MultipartResolver MultipartResolver() {
		CommonsMultipartResolver resolver = new CommonsMultipartResolver();
		resolver.setDefaultEncoding("UTF-8");
		resolver.setMaxUploadSize(1048576);		//总的文件上传大小
		return resolver;
	}
}
```
控制类
```java
@Controller
public class WopiController {

    @Autowired
	private Environment env;

    @RequestMapping(value="/upload", method=RequestMethod.POST)
	public void putFile(@RequestParam("file")MultipartFile multipartFile) {
        try {
            String filePath = Paths.get(env.getRequiredProperty("tmp.dir"), multipartFile.getOriginalFilename()).toString();
            File file = new File(filePath);
            multipartFile.transferTo(file);
        } catch (IllegalStateException | IOException | FileOperationException e) {
            e.printStackTrace();
        }
    }
}
```
测试类
```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(classes = { WebMvcConfig.class })
public class TestController {
    @Autowired
	private WebApplicationContext wac;
	private MockMvc mockMvc;

    @Before
	public void setup() {
		this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }

    @Test
	public void testPutFile() throws Exception {
		String uploadPath = "C:/test1.xls";
		MockMultipartFile file = new MockMultipartFile("file", new FileInputStream(uploadPath));
		mockMvc.perform(fileUpload("/upload").file(file)).andExpect(status().isOk());
	}
}
```

----

### 文件下载

```java
@RequestMapping(value="/files/{fileId}/contents", method=RequestMethod.GET)
public void getFile(@PathVariable String fileId,  HttpServletResponse response) {
    String filePath = wopiService.getFilePath(fileId);
    Path path = Paths.get(filePath);
    if (Files.exists(path, LinkOption.NOFOLLOW_LINKS)) {
        try {
            File file = path.toFile();
            response.setContentType("application/octet-stream");
            response.setContentLength((int)file.length());
            response.setHeader("Content-Disposition", "attachment;filename=\"" + file.getName() + "\"");
            InputStream inputStream = new BufferedInputStream(new FileInputStream(file));
            FileCopyUtils.copy(inputStream, response.getOutputStream());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
