泥瓦匠网盘资源教程:

 https://pan.baidu.com/s/1zZ3lLhvaBkTBIl7zTZGycw 提取码: r8zg

#### 参考教程

Spring Boot 系列：
http://www.spring4all.com/article/246

Spring Cloud 系列：
http://www.spring4all.com/article/320

微服务架构系列：
http://www.spring4all.com/article/609

 

#### 项目结构

![1555573559987](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1555573559987.png)





#### 配置参数设定

Spring Boot 不单单从 application.properties 获取配置，所以我们可以在程序中多种设置配置属性。按照以下列表的优先级排列：

1.命令行参数

2.java:comp/env 里的 JNDI 属性

3.JVM 系统属性

4.操作系统环境变量

5.RandomValuePropertySource 属性类生成的 random.* 属性

6.应用以外的 application.properties（或 yml）文件

7.打包在应用内的 application.properties（或 yml）文件

8.在应用 @Configuration 配置类中，用 @PropertySource 注解声明的属性文件

9.SpringApplication.setDefaultProperties 声明的默认属性

注：

**中文配置参数读取乱码问题**

一个是**配置文件编码方式**不是`UTF-8`的，另一个是`Spring http`使用的编码不是`UTF-8`。

1.Spring/SpringBoot只能用Properties类解析读取properties文件，而该文件默认使用ISO字符编码，因此会导致配置读入乱码问题。解决方式：在setter方法中使用new String(xxx.getBytes(StandardCharsets.ISO_8859_1), StandardCharsets.UTF_8)读取再重新set一次。

#### 打包部署

1.在pom文件中的<build>标签中指定主类，否则会报错。导入maven插件，配置主类名

![image.png](https://upload-images.jianshu.io/upload_images/5548226-3249288d08ff1638.png?imageMogr2/auto-orient/strip)

```
<!--解决SpringBoot打包成jar后运行提示没有主清单属性-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

2.让启动类继承SpringBootServletInitializer

然后使用右边栏maven工具先clean然后package或者install即可



#### 单元测试

添加依赖：

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-test</artifactId>
   <scope>test</scope>
</dependency>
```

注：不要添加springboot-test依赖包，会导致冲突

添加注解：

```
@RunWith(SpringRunner.class)
@SpringBootTest
```

Controller测试：进行Mock模拟web请求测试时，需要注解

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
public class MockApplicationTests {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception{
        mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
    }
```

**SpringMVC单元测试**

```
/**
 * 使用Spring测试模块提供的测试请求功能，测试curd请求的正确性
 * Spring4测试的时候，需要servlet3.0的支持
 * @author lfy
 *
 */
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "classpath:applicationContext.xml",
      "file:src/main/webapp/WEB-INF/dispatcherServlet-servlet.xml" })
public class MvcTest {
   // 传入Springmvc的ioc
   @Autowired
   WebApplicationContext context;
   // 虚拟mvc请求，获取到处理结果。
   MockMvc mockMvc;

   @Before
   public void initMokcMvc() {
      mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
   }

   @Test
   public void testPage() throws Exception {
      //模拟请求拿到返回值
      MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get("/emps").param("pn", "5"))
            .andReturn();

      //请求成功以后，请求域中会有pageInfo；我们可以取出pageInfo进行验证
      MockHttpServletRequest request = result.getRequest();
      PageInfo pi = (PageInfo) request.getAttribute("pageInfo");
      System.out.println("当前页码："+pi.getPageNum());
      System.out.println("总页码："+pi.getPages());
      System.out.println("总记录数："+pi.getTotal());
      System.out.println("在页面需要连续显示的页码");
      int[] nums = pi.getNavigatepageNums();
      for (int i : nums) {
         System.out.print(" "+i);
      }

      //获取员工数据
      List<Employee> list = pi.getList();
      for (Employee employee : list) {
         System.out.println("ID："+employee.getEmpId()+"==>Name:"+employee.getEmpName());
      }

   }

}
```

**Mybatis单元测试**

在SSM中

```
/**由于测试mybatis事务需要注入dao，service，因此需要web**/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:spring.xml","classpath:spring-mybatis.xml"})
@Test
//进行mybatis事务测试
@Rollback(value = false)
public void testMybatis()throws Exception{


    City city = new City();
    city.setCityName("重庆");
    city.setProvinceId(2l);
    city.setDescription("我的家乡");
    //返回影响的条目数
    logger.info(cityDao.saveCity(city).toString());

}
```

在Springboot中

```
@RunWith(SpringRunner.class)
@MybatisTest
//使用自定义配置的数据源,否则使用虚拟数据源
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class MybatisTests {

    @Autowired
    private CityDao cityDao;


    private static Logger logger = LoggerFactory.getLogger(MybatisTests.class);


    @Test
    //进行mybatis事务测试
    @Rollback(value = false)
    public void testMybatis()throws Exception{


        City city = new City();
        city.setCityName("重庆");
        city.setProvinceId(2l);
        city.setDescription("我的家乡");
        //返回影响的条目数
        logger.info(cityDao.saveCity(city).toString());

    }
}
```



#### 注解

@RestContrller：Spring4之前返回json需要配合@responseBody，使用该注解可以直接返回json

@RequestParam:实质是将Request.getParameter() 中的**Key-Value参数Map利用Spring的转化机制ConversionService配置，转化成参数接收对象或字段。**

@ModelAttribute:绑定参数

**区别：**

1、@RequestParam，@RequestParam("xx") 表示在前端传递过来的参数中**必须有个参数名称为“xx”**（可以使用required=false避免必须）。**其可以自动转换为基本类型。**

2、@ModelAttribute，@ModelAttribute("xx") 表示将前端传递过来的参数**按照名称注入到对应的对象**中，“xx”只是表示放到ModelMap中的key值。也就是说，ModelAttribute是在传递时提前在Model域中**实例化了一个空对象，收到数据后调用setter进行注入。**

3.application/json、application/xml等格式的数据，**必须使用@RequestBody来处理**

@PathVariable：用于绑定url上的参数，但不能

@RequestBody:用于接收post请求的json格式的数据，注入到参数的实体类型中或者map类型

@InitBinder：可以对web传过来的数据进行初始化。即对 WebDataBinder 对象进行初始化。WebDataBinder 是 DataBinder 的子类,**用于完成由表单字段到 JavaBean 属性的绑定**。例如在controller中定义：

	@InitBinder  
	protected void initBinder(WebDataBinder binder) {  
	    binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"), true));  
	    binder.registerCustomEditor(int.class, new CustomNumberEditor(int.class, true));  
	    binder.registerCustomEditor(int.class, new IntegerEditor());  
	    binder.registerCustomEditor(long.class, new CustomNumberEditor(long.class, true));
	    binder.registerCustomEditor(long.class, new LongEditor());  
	    binder.registerCustomEditor(double.class, new DoubleEditor());  
	    binder.registerCustomEditor(float.class, new FloatEditor());  
	    } 
@RequestHeader:获取请求头中的字段值，例如@RequestHeader("Accept-Encoding") String encoding

@CookieValue：获取cookie值并绑定到方法参数，例如@CookieValue("JSESSIONID") String cookie

@WebAppCofiguration : 指定加载 ApplicationContext是一个WebApplicationContext,实现MockMVC测试的构造参数。

#### Swagger2

它可以轻松的整合到Spring Boot中，并与Spring MVC程序配合组织出强大RESTful API文档。它既可以减少我们创建文档的工作量，同时说明内容又整合入实现代码中，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明。另外Swagger2也提供了强大的页面测试功能来调试每个RESTful API。具体效果如下图所示：

![alt=](http://upload-images.jianshu.io/upload_images/1447174-8293829f8bd18e45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### springboot启动原理

![img](https://mmbiz.qpic.cn/mmbiz_png/nXyTmFfqCEM7WFvQ0Sd3AZJbuEhw6pLbbevdI6LcZr2jTzyDbkZFjgZA9282Zc2BzGOByYC5OCadialICliajnNw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们将各步骤总结精炼如下：

1. 通过 `SpringFactoriesLoader` 加载 `META-INF/spring.factories` 文件，获取并创建 `SpringApplicationRunListener` 对象
2. 然后由 `SpringApplicationRunListener` 来发出 starting 消息
3. 创建参数，并配置当前 SpringBoot 应用将要使用的 Environment
4. 完成之后，依然由 `SpringApplicationRunListener` 来发出 environmentPrepared 消息
5. 创建 `ApplicationContext`
6. 初始化 `ApplicationContext`，并设置 Environment，加载相关配置等
7. 由 `SpringApplicationRunListener` 来发出 `contextPrepared` 消息，告知SpringBoot 应用使用的 `ApplicationContext` 已准备OK
8. 将各种 beans 装载入 `ApplicationContext`，继续由 `SpringApplicationRunListener` 来发出 contextLoaded 消息，告知 SpringBoot 应用使用的 `ApplicationContext` 已装填OK
9. refresh ApplicationContext，完成IoC容器可用的最后一步
10. 由 `SpringApplicationRunListener` 来发出 started 消息
11. 完成最终的程序的启动
12. 由 `SpringApplicationRunListener` 来发出 running 消息，告知程序已运行起来了