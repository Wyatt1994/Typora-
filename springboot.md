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

进行Mock模拟web请求测试时，需要注解

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

#### 注解

@RestContrller：Spring4之前返回json需要配合@responseBody，使用该注解可以直接返回json

@RequestParam:实质是将Request.getParameter() 中的**Key-Value参数Map利用Spring的转化机制ConversionService配置，转化成参数接收对象或字段。**

@ModelAttribute:绑定参数

**区别：**

1、@RequestParam，@RequestParam("xx") 表示在前端传递过来的参数中**必须有个参数名称为“xx”**（可以使用required=false避免必须）

2、@ModelAttribute，@ModelAttribute("xx") 表示将前端传递过来的参数**按照名称注入到对应的对象**中，“xx”只是表示放到ModelMap中的key值。也就是说，ModelAttribute是在传递时提前在Model域中**实例化了一个空对象，收到数据后调用setter进行注入。**

@PathVariable：用于绑定url上的参数