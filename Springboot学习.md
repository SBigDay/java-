# Springboot学习

## 一.注解

### 2.web层

#### 1.@RestController 

1.@RestController 直接返回json格式

#### 2.@RequestMapping @PathVariable @RequestParam @GetMapping

1.@RequestMapping(“url地址”)

```
@RequestMapping(value = "/hello")

public String hello(@RequestParam(value = "name") String name);
```

2.@PathVariable和 @RequestParam的区别

@RequestParam注解是获取静态URL传入的参数

@PathVariable是获取请求路径中的变量作为参数

    @RequestMapping(value = "/list", method = {RequestMethod.GET, RequestMethod.POST})
    public Map<String, Object> list(@RequestParam Long user_id) {
    		Map<String, Object> map = new HashMap<String, Object>();
    		map.put("user_id", user_id);
        return map;
    }
    @RequestMapping(value = "/get_user/{user_id}", method = {RequestMethod.GET, RequestMethod.POST})
    public Map<String, Object> get_user(@PathVariable Long user_id) {
    		Map<String, Object> map = new HashMap<String, Object>();
    		map.put("user_id", user_id);
        return map;
    }

3.@GetMapping（“url地址”） 组合注解  HTTP GET形式

是@RequestMapping(method = RequestMethod.GET)的缩写。

#### 3.@RequestParam和@Requestbody

   如果参数时放在请求体中，传入后台的话，那么后台要用@RequestBody才能接收到;如果不是放在请求体中的话，那么后台接收前台传过来的参数时，要用@RequestParam（）来接收，或则形参前什么也不写也能接收。

#### 4.@Qualifier

用于区分一个servcie的两个实现类

在Autowired注入时，用这个注解标注。不只是service层，哪个地方需要标注都可以用这个注解。

```
 @Bean
 @Qualifier(RabbitMQConstant.PROGRAMMATICALLY_EXCHANGE)
 TopicExchange exchange() {
     return new TopicExchange(RabbitMQConstant.PROGRAMMATICALLY_EXCHANGE, false, true);
    }
```

### 3.实体类注解

#### @Component()

把pojo对象注入到spring容器中 相当于<bean id="" class=""/>）

@Component 注解表面该类会作为组建类。并告知spring要为这个类创建bean ，需要注意的是组建扫描默认是不开启的。需要配置扫描，让扫描组建去扫描带有@Component的注解   

@Configuration用于全局配置，比如数据库相关配置，MVC相关配置等；业务Bean的配置使用注解配置(@Component,@Service,@Repository,@Controller)。

#### @ComponentScan

就是扫描组建，默认情况下该注解只会扫描同包中的注解。

但是SpringbootApplication中集成了@SpringBootConfiguration@ComponentScan注解 所以这个完全不用写。

#### @Entity    

@Entity    hibernate声明这是一个实体类 字段名称应与数据库中的名字对应上

#### @Column注解

用来标识实体类中属性与数据表中字段的对应关系unique=true是指这个字段的值在这张表里不能重复，所有记录值都要唯一，就像主键那样;nullable=false是这个字段在保存时必需有值，不能还是null值就调用save去保存入库; 类

#### @ApiModelProperty  用于标注字段作用，名称啥的

#### @Data        

可以不用写构造方法，get,set等

```
<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
```

#### @Configurable

不是基于Spring的应用程序中使用的所有类型(bean)都是Spring管理的。有时，我们需要将一些依赖注入到一些非Spring管理的类型中。最常见的例子是在实体类中注入服务或DAO层。现在实体类的实例使用**new**创建，也即它们不是被Spring管理的。现在我们想要在这个类型中注入对应的Service或DAO类型，被注入类型是被Spring管理的，以上这些操作可以在 **@configure** 的帮助下完成。

#### @Autowired

将实例bean注册到spring容器，springbootApplication将bean扫描到了之后，就可以用@Autowired实例一个了。

#### @EnableScheduling

标注启动定时任务

### 4.微服务注解

#### @EnableEurekaServer() ：

一般用在注册中心

#### @EnableFeignClients：

启用feign进行远程调用这个注解放在启动类上

#### @FeignClients

@FeignClients(name=”spring-cloud-producer”)

用在某一个接口上

#### @EnableDiscoveryClient 

将一个服务注册到eureka中

###  5.枚举

#### @Enumerated

(**EnumType**.**STRING**)

#### **@Column**

(**nullable **=** **true****)使用枚举

### 6.事务

#### @Transient

修饰不需要和数据库关联的关键字

#### @Transactional

标注事务

###  7.缓存注解

#### @cacheable

### 8.JPA

jdbc->mybatis->hebernate->jpa

@Id 标注用于声明一个实体类的属性映射为数据库的主键列        

@GeneratedValue 用于标注主键的生成策略，通过strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略

SpringBoot的@GeneratedValue 是不需要加参数的,但是如果数据库控制主键自增(auto_increment), 不加参数就会报错.   

### 9.@Configuration和@bean

@Configuration用于全局配置，比如数据库相关配置，MVC相关配置等；业务Bean的配置使用注解配置(@Component,@Service,@Repository,@Controller)。

从Spring3.0，@Configuration用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

**注意**：@Configuration注解的配置类有如下要求：

1. @Configuration不可以是final类型；

2. @Configuration不可以是匿名类；

3. 嵌套的configuration必须是静态类。

   SpringbootApplication中集成了@SpringBootConfiguration所以不需要Context扫描

   @Configuration和@bean组合使用 @Configuration修饰类，@bean修饰方法

   ```
   @Configuration
   public class TestConfiguration {
       public TestConfiguration() {
           System.out.println("TestConfiguration容器启动初始化。。。");
       }
   
       // @Bean注解注册bean,同时可以指定初始化和销毁方法
       // @Bean(name="testBean",initMethod="start",destroyMethod="cleanUp")
       @Bean
       @Scope("prototype")
       public TestBean testBean() {
           return new TestBean();//这里的TestBean()是随便一个实体类
       }
   }
   ```

   ```
   public class TestMain {
       public static void main(String[] args) {
   
           // @Configuration注解的spring容器加载方式，用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext
           ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
   
           // 如果加载spring-context.xml文件：
           // ApplicationContext context = new
           // ClassPathXmlApplicationContext("spring-context.xml");
           
            //获取bean
           TestBean tb = (TestBean) context.getBean("testBean");
           tb.sayHello();
       }
   }
   ```

   

### 10.mapper层

#### 1.@Repository

用于标识数据访问组件,一般在mapper层使用

但是启动类必须得有@MapperScan扫描，或者把@Repository换成@mapper

#### **2.@Param  ** 

当mapper层需要传多个参数时 必须使用@Param注解 标注每一个参数

**1.注解单一属性**

**注解javabean对象**

[**https://blog.csdn.net/u012031380/article/details/54924641**](https://blog.csdn.net/u012031380/article/details/54924641)

**在别的层注解****@MapperScan****直接通过路径访问mapper不用写@mapper了**

#### 3.@mapper  

属于mybatis的注解，并不是spring的 所以要使mapper能被其他类引用，就得加@mapper注解

这个注解就是为了不用写mapper.xml文件，直接将dao交给spring处理

```
/**
 * 添加了@Mapper注解之后这个接口在编译时会生成相应的实现类
 * 
 * 需要注意的是：这个接口中不可以定义同名的方法，因为会生成相同的id
 * 也就是说这个接口是不支持重载的
 */
@Mapper
public interface UserDAO {

    @Select("select * from user where name = #{name}")
    public User find(String name);

    @Select("select * from user where name = #{name} and pwd = #{pwd}")
    /**
      * 对于多个参数来说，每个参数之前都要加上@Param注解，
      * 要不然会找不到对应的参数进而报错
      */
    public User login(@Param("name")String name, @Param("pwd")String pwd);
}
```

#### 4.@mapperScan

如果不想在每一个mapper上都加@mapper可以在启动类上加一个@mapperScan注解

#### 5.mapper.java和xml不在一个包下

 application。properties下应写

```
mybatis.type-aliases-package=com.example.schoolfinnal.mapper
mybatis.mapper-locations= classpath:mapper/**/*.xml
```



### 11.**swagger**层远程调用

**@Api用在controller层           协议集描述**

**@ApiModel 用在返回类对象上         描述返回对象的意义**

**@ApiModelProperty**    **修饰对应属性**   

**https://blog.csdn.net/chinassj/article/details/81875038**

### 12.**Rpc层**

**是一个框架，调用远程代码就像调用本地代码一样，很神奇。里面使用了动态代理**

### 13**vo层    应该是启动类**

*//**VO*对应于页面上需要显示的数据（表单），*DO*对应于数据库中存储的数据（数据表），*DTO*对应于除二者之外需要进行传递的数据。

### 

### 14service层 

#### @Value

引用配置文件中的内容，例如yml文件

```
spring:
    profiles: dev

user:
    userName: 王立国-dev
    sex: 男
    age: 18
author:
    name: wlg-dev
    height: 174
```

```
@Service
public class HelloWorldServiceImpl implements IHelloWorldService {

    @Autowired
    private MyConfig myConfig;

    @Value(value = "${user.userName}")
    private String userName;
    @Value("${user.sex}")
    private String sex;
    @Value("${user.age}")
    private String age;

    @Override
    public String getMessage() {
        return "姓名："+userName+" 性别："+sex+" 年龄："+ age +" 作者&身高："+myConfig.getName()+" "+myConfig.getHeight();
    }
}

```

### 15.Biz层

#### @Scheduled 定时任务

```
@Scheduled(cron = "*/5 * * * * ?")    每隔5秒执行一次
```

#### @Resource 和@Autowired

两个作用相似，都是依赖注入

## 

## 二．Thymeleaf前端的使用了解

```java
Url:<a th:href="@{http://www.thymeleaf.org}">Thymeleaf</a>
条件求值：<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
for循环<tr th:each="prod : ${prods}">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td></tr>
```

http://www.ityouknow.com/springboot/2016/02/03/spring-boot-web.html



# 







