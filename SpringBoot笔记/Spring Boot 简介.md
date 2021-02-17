

# 一、Spring Boot 简介

​	1.简化Spring应用开发的一个框架；

​	2.整个Spring技术栈的一个大整合

​	3.J2EE开发的一站解决方案



```xml
		<plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>

```

这个插件可以将项目打包成一个可执行的jar包  通过 java -jar +jar 包名字 前提要先进入jar包的具体位置

打成jar包步骤   1.点击Maven下找 Lifecycle目录  点击 package 就行

### 1、pom文件深究

```xml
pom文件引入了一个父项目 
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
父项目引用了dependencies父项目
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.1</version>
  </parent>
该父文件里有很多的版本号 意味着以后导jar包都不用写版本号（里面没有的要自己写）
```



### 2、启动器



```xml
 	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
点进去可以查看具体的jar包
这个启动器里导入了很多的jar包 Spring Boot帮我们导入的 向这种启动器官网上还有很多（不过我没找到）

```



### 3、@SpringBootApplication详解怎么工作的

#### 1.@SpringBootConfiguration 

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

@SpringBootConfiguration注解翻译过来就是 Spring Boot的配置 @SpringBootApplication 中所有的配置都在这个注解里面

@SpringBootConfiguration下有一个@Configuration 详细配置注解 

#### 2、@EnableAutoConfiguration

@EnableAutoConfiguration注解翻译过来就是自动配置  Spring Boot的所有自动配置都在里面

点进去以后发现@Import(AutoConfigurationImportSelector.class)  这句注解表示导入AutoConfigurationImportSelector.class 自动装配类

点进去发现一个selectImports(AnnotationMetadata annotationMetadata)方法  返回的一个字符串数组

点击 getAutoConfigurationEntry方法

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return NO_IMPORTS;
   }
   AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
   return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
   }
   AnnotationAttributes attributes = getAttributes(annotationMetadata);
   List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   configurations = removeDuplicates(configurations);
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   checkExcludedClasses(configurations, exclusions);
   configurations.removeAll(exclusions);
   configurations = getConfigurationClassFilter().filter(configurations);
   fireAutoConfigurationImportEvents(configurations, exclusions);
   return new AutoConfigurationEntry(configurations, exclusions);
}
 List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   configurations = removeDuplicates(configurations);
这两行代码通过debug方式得知 所有的AutoConfig配置文件都在这里面。点进去发现能找到配置文件的具体位置
    
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

点击 loadFactoryNames 发现该方法里有 这个FACTORIES_RESOURCE_LOCATION点进去发现了配置文件位置
    Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);

	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

找到相应的jar包
    EnableAutoConfiguration可以找到
```



### 4、yaml与xml 和 properties配置文件的区别

#### 1、yaml

yaml可以简写为 yml  书写简单 建议使用

```xml
server:
  port: 8081
```

properties配置文件不易于表达

```xml
server.port=8081
```

xml书写麻烦

```xml
<server>
	<port>8081</port>
</server>
```



### 5、yaml语法

```xml

account:
  username: 小红
  age: 18
  boos: false
  birth: 2018/10/1
  maps: {k1: v1,k2: v2}
  list:
    - 小鬼
    - 小黑
  dog:
    username: xaio
    age: 19
```

实体类

```xml
@Component
@ConfigurationProperties(prefix = "account")   //告诉配置文件叫啥
public class account {
    private String username;
    private Integer age;
    private boolean boos;
    private Date birth;
    private Map maps;
    private List list;
    private Dog dog;
导入这个jar包以后写 yaml文件会有提示
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

测试类

```xml
//用Spring的驱动器来跑而不是用junit
//SpringRunner
@RunWith(SpringRunner.class)
@SpringBootTest   //告诉他这个是一个SpringBoot的东西
class DemoApplicationTests {
   @Autowired
    account account;
    @Test
    void contextLoads() {
        System.out.println(account);
    }
```



### 6、@PropertySource(value = "classpath:person.properties")

@PropertySource(value = "classpath:person.properties") 告诉SpringBoot要用这个文件中的东西

```java
@Component
@PropertySource(value = {"classpath:person.properties"})  用person.properties注入
@ConfigurationProperties(prefix = "account")   使用yaml文件注入值
public class account {
//    @Value("${account.user-name}")
    private String userName;
//    @Value("#{10*2}")
    private Integer age;
    private Boolean boos;
    private Date birth;
    private Map maps;
    private List list;
    private Dog dog;


```



### 7、@ImportResource

@ImportResource(locations = {"classpath:bean.xml"})  将让SpringBoot读取到Spring的配置文件bean.xml

```xml
项目启动的上面加上
@ImportResource(locations = {"classpath:bean.xml"})
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

测试类
   @Autowired
   ApplicationContext ioc;
   @Test
//判断ioc中是否包含dao
//重要方法containsBean 跟集合中的包含一摸一样
   void a(){
       boolean account = ioc.containsBean("dao");
       System.out.println(account);
   }

bean.xml
 <bean id="dao" class="com.example.demo.dao.dao"></bean>
```



### 8、Spring Boot支持的装配Spring配置文件的方法

​	通过@Bean注解的方式装配Spring的配置文件

```java
@Configuration   //声明这是一个配置类
public class service {
    @Bean       //配置Bean   这个配置类相当于 xml文件中的    
    	// <bean id="serviceDao" class="com.example.demo.dao.dao"></bean>
    public dao serviceDao(){
        System.out.println("我准备装配dao了");
        return new dao();
    }
}


测试类
    
    @Autowired  
   ApplicationContext ioc;
     @Test
   void b(){
       boolean serviceDao = ioc.containsBean("serviceDao");
       System.out.println(serviceDao);
   }
```

### 9、默认值跟占位符



```properties
account:
  user-name: ${random.value}小红    //${random.value}随机数
  age: ${random.int}			//${random.int}随机数
  boos: false
  birth: 2018/10/1
  maps: {k1: v1,k2: v2}
  list:
    - 小鬼-${account.user-name:123}  //${account.user-name:123}占位符 没找到的话：123就是默认值
    - 小黑
  dog:
    username: xaio
    age: 19
```

### 10、配置文件的优先级

​	-file:./config/ 在项目根目录下写一个config里的配置文件优先级最高

​	-file:./项目根目录下写直接写配置文件优先级第二

​	-classpath:/config/ 在resources目录下创建一个config 中的配置文件优先级第三

​	-classpath:/  在resources目录下不写包配置文件优先级最低



### 11、SpringBoot的自动配置源码分析

以一个简单的为例   以HttpEncodingAutoConfiguration 类为例

```java
@Configuration(proxyBeanMethods = false) //添加配置 将这个类变成一个配置类
@EnableConfigurationProperties(ServerProperties.class) //指的是启动类 启动ServerProperties 类 绑定properties  并把ServerProperties加入到ioc容器中(简单的说就是自动配置)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)//判断是不是web应用
@ConditionalOnClass(CharacterEncodingFilter.class)//判断当前项目有没有这个类
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
//判断当前文件是不有有指定的配置  server.servlet.encoding.enabled ；如果不存在判断也成立   因为matchIfMissing这个东西
public class HttpEncodingAutoConfiguration {
    private final Encoding properties;//已经和SpringBoot的配置文件映射起来了
    public HttpEncodingAutoConfiguration(ServerProperties properties) {//通过有参构造，参数的值就会从ioc容器中拿
		this.properties = properties.getServlet().getEncoding();
	}
    //以上条件都成立就执行以下条件
    @Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name()); //获取
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}
```



#### 总结自动装配的原理

1.SpringBoot启动会加载大量的自动配置类

2.我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类中

3.我们再来看这个自动配置类中到底配置了那些组件；（只要我 们需要的组件在其中我们就不需要在手动配置了）

4.给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可；

xxxxAutoconfigurartion:自动配置类；给容器添加组件

xxxxproperties:封装配置文件中相关属性;







### 12、@Conditional派生注解详解

```xml
@Conditional                                       	| 作用（判断是否满足当前指定条件）              
@ConditionalOnJava                           	 	| 系统的java版本是否符合要求               
@ConditionalOnBean                          | 容器中存在指定Bean；                  
@ConditionalOnMissingBean              | 容器中不存在指定Bean；                 
@ConditionalOnExpression                 | 满足SpEL表达式指定                   
@ConditionalOnClass                          | 系统中有指定的类                      
@ConditionalOnMissingClass              | 系统中没有指定的类                     
@ConditionalOnSingleCandidate         | 容器中只有一个指定的Bean，或者这个Bean是首选Bean
@ConditionalOnProperty                      | 系统中指定的属性是否有指定的值               
@ConditionalOnResource                    | 类路径下是否存在指定资源文件                
@ConditionalOnWebApplication           | 当前是web环境                      
@ConditionalOnNotWebApplication      | 当前不是web环境                     
@ConditionalOnJndi                              | JNDI存在指定项     
```

自动配置类中并不是所有的配置都用上了  有很多不满足条件的配置类都没有启动可以通过 Debug=true的方式进行查看

```java
以下都是用到了的自动配置类
Positive matches:
-----------------
   AopAutoConfiguration matched:
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.ClassProxyingConfiguration matched:
      - @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet' (OnClassCondition)
      - found 'session' scope (OnWebApplicationCondition)
  
          
   以下是没有用到的自动配置类
 Negative matches:
-----------------
   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration.AspectJAutoProxyingConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice' (OnClassCondition)

   ArtemisAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

```



### 13、SpringBoot日志

```java
Logger logger = LoggerFactory.getLogger(getClass());
//默认从级别最高的地方开始一直输出到级别最低的 SpringBoot默认root是 info 所以只输出info以下的日志
//想改级别去配置文件改   你想要的配置文件都有
@Test
void contextLoads() {
    logger.trace("我是trace级别最高哦...");
    logger.debug("俺是Debug 级别第二");
    logger.info("i is info 级别第三");
    logger.warn("爸爸是 warn 级别第四");
    logger.error("爷是错误error级别第五");
}
properties配置文件
#级别设置为最高  所有的日志都打印
logging.level.com.lsm=trace 

#日志打印在file文件中
#logging.file.name=logging.log
#指定路径打印到指定路径中
#logging.file.name=D:logging.log

```



# 二、Web开发

### 1、SpringBoot对静态资源的映射元源码分析

主要分析  WebMvcAutoConfiguration 类

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
      ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

#### (1) 当资源是外部导入时



```java
@Override
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
   super.addResourceHandlers(registry);
   if (!this.resourceProperties.isAddMappings()) {
      logger.debug("Default resource handling disabled");
      return;
   }
   ServletContext servletContext = getServletContext();
   addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
   addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
      registration.addResourceLocations(this.resourceProperties.getStaticLocations());
      if (servletContext != null) {
         registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
      }
   });
}
```

通过 https://www.webjars.org/documentation 网站导入maven的jar包 

```xml
webjars的包jquery
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

由以上源码的知  SpringBoot会自动访问  "/webjars/**"   "classpath:/META-INF/resources/webjars/"

```java
addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
```

访问路径 http://localhost:8080/webjars/jquery/3.5.1/jquery.js   

#### (2)当资源在本地时

```java
@Override
		protected void addResourceHandlers(ResourceHandlerRegistry registry) {
			super.addResourceHandlers(registry);
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			ServletContext servletContext = getServletContext();
			addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
			addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
				registration.addResourceLocations(this.resourceProperties.getStaticLocations());
				if (servletContext != null) {
					registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
				}
			});
		}
```



getStaticPathPattern方法返回 "/**" 获取所有路径下的所有资源  如果没人处理则getStaticLocations 

getStaticLocations返回一个String[ ] 如果没人处理的话就会访问这数组中的路径   路径在下面

```java
"classpath:/META-INF/resources/", //类路径下创建一个/META-INF/resources/ 放静态资源文件
"classpath:/resources/", //类路径下创建一个/resources/ 放静态资源文件
"classpath:/static/", 	//类路径下创建一个static 放静态资源文件
"classpath:/public/" 	//类路径下创建一个public 放静态资源文件
    优先级  resources>static>public
```

具体如图所示

![](E:\SpringBoot笔记\images\a.PNG)

#### （3）怎么找到html首页的

```java
private Resource getWelcomePage() {
    //this.resourceProperties.getStaticLocations()默认资源的路径  循环手游的资源路径 找完了看能不能找到
   for (String location : this.resourceProperties.getStaticLocations()) { 
       //调用 getIndexHtml（）方法 
      Resource indexHtml = getIndexHtml(location);
      if (indexHtml != null) {
         return indexHtml;
      }
   }
   ServletContext servletContext = getServletContext();
   if (servletContext != null) {
      return getIndexHtml(new ServletContextResource(servletContext, SERVLET_LOCATION));
   }
   return null;
}
//来到这里
private Resource getIndexHtml(String location) {
    //调用getIndexHtml（）资源路径传进去
   return getIndexHtml(this.resourceLoader.getResource(location));
}
//来到这里 从默认资源路径找到index.heml
private Resource getIndexHtml(Resource location) {
   try {
      Resource resource = location.createRelative("index.html");
      if (resource.exists() && (resource.getURL() != null)) {
         return resource;
      }
   }
   catch (Exception ex) {
   }
   return null;
}
```



#### (4)yaml优先级位置关系

```xml
1. file:./config/  一
2. file:./		二
3. classpath:/config/	三
4. classpath:/	四
```

![](E:\SpringBoot笔记\images\d.PNG)



### 2、彩蛋

将文件取名为 favicon.ico 放在任何资源目录下都能用

作用 像Vue 的图标一样在浏览器前面显示   如图所示

![](E:\SpringBoot笔记\images\b.PNG)



### 3、Thymeleaf  模板引擎

####  （1）Thymeleaf  模板引擎特点

   - 动静结合 ：Thymeleaf在有网没网的环境下都能运行，即他可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器上查看带数据的动态页面效果。这是由于他的html原型然后在html标签里增加额外的属性来达到模板+数据展示方式。在浏览器解释html时会忽略未定义的标签属性，所以Thymeleaf的模板可以静态的运行；当有数据返回页面时，Thymeleaf标签会动态的替换掉静态内容，使页面动态显示。

   - 开箱即用：他提供标准和Spring标准两种方言，可以直接套用模板实现JSTL、OGNL表达式效果避免每天套模板、该jstl、改标签的困扰。同事开发人员也可以扩展和创建自定义的方言。

   - 多方言支持：Thymeleaf 提供spring标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能

   - 与SpringBoot完美整合，SpringBoot提供了Thymeleaf的默认配置，并且为Thymeleaf设置了视图解析器，我们可以像以前操作jsp一样来操作Thymeleaf。代码几乎没有任何区别，就是在模板语法上有区别

     导入的jar包

     ```xml
     <dependencies>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-thymeleaf</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-web</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-test</artifactId>  //版本号SpringBoot已经配置好了
             </dependency>
         </dependencies>
     ```

Spring-Boot-autoconfigure  已经配置好了视图解析器  具体位置如下：

![](E:\SpringBoot笔记\images\c.PNG)

视图解析器具体代码如下： 其实跟SpringMVC的视图解析器差不多  静态文件必须放在templates包下 并且要以.heml后缀结尾 

ThymeleafProperties

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

   private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

   public static final String DEFAULT_PREFIX = "classpath:/templates/";

   public static final String DEFAULT_SUFFIX = ".html";
```



#### （2）具体语法

controller页面代码

```java
@Controller
public class HelloController {
    @RequestMapping("/xxoo")
    public String xxoo(Map<String,Object> map){
        map.put("hello","<h1>你好</h1>");
        map.put("session","<h1>你好</h1>");
        map.put("list", Arrays.asList("大壮","小强","李青"));
        return "hello";
    }
}
```



html页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">    heml加上mlns:th="http://www.thymeleaf.org" 开启提示
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div th:text="${hello}"></div>    th：text 输出数据 可以放在任何标签任何属性上使用   后台数据有标签的话代标签输出
<div th:text="${hello}"></div>
<div th:utext="${hello}"></div>	th:utext后台传过来的标签都能直接用
<div th:text="${list}" th:each="list:${list}"></div>   相当于for循环
    
    以下是在网上扣得自己没有实现  因为懒
    <h2>
    <p>Name: <span th:text="${user.name}">Jack</span>.</p>   el表达式差不多
    <p>Age: <span th:text="${user.age}">21</span>.</p>
    <p>friend: <span th:text="${user.friend.name}">Rose</span>.</p>  
	</h2>
    
    <h2 th:object="${user}">
    <p>Name: <span th:text="*{name}">Jack</span>.</p>   作对比自己找不同
    <p>Age: <span th:text="*{age}">21</span>.</p>
    <p>friend: <span th:text="*{friend.name}">Rose</span>.</p>
	</h2>
</body>
</html>
```

```properties
thymeleaf所有的语法
日期格式化 
"${#dates.format(xx.xx,'yyy-mm-dd hh: mm')}"
th:if="${not #strings.isEmpty(msg)}"  判断msg是否为空 空的话不显示  not取反 优先级第三
th:checked="${xx.xx==1}"  用于单选框  可以做判断 true则选中
th:selected="${xxx.xx=xxx.xx}" 用于下拉框 true选中
th:value="${xx.xx}" 跟value一样 input赋值用的
th:id 	替换id 	<input th:id="'xxx' + ${collect.id}"/>
th:text 	文本替换 	<p th:text="${collect.description}">description</p>
th:utext 	支持html的文本替换 	<p th:utext="${htmlcontent}">conten</p>
th:object 	替换对象 	<div th:object="${session.user}">
th:value 	属性赋值 	<input th:value="${user.name}" />
th:with 	变量赋值运算 	<div th:with="isEven=${prodStat.count}%2==0"></div>
th:style 	设置样式 	th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''"
th:onclick 	点击事件 	th:onclick="'getCollect()'"
th:each 	属性赋值 	tr th:each="user,userStat:${users}">
th:if 	判断条件 	<a th:if="${userId == collect.userId}" >
th:unless 	和th:if判断相反 	<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
th:href 	链接地址 	<a th:href="@{/login}" th:unless=${session.user != null}>Login</a> />
th:switch 	多路选择 配合th:case 使用 	<div th:switch="${user.role}">
th:case 	th:switch的一个分支 	<p th:case="'admin'">User is an administrator</p>
th:fragment 	布局标签，定义一个代码片段，方便其它地方引用 	<div th:fragment="alert">
th:include 	布局标签，替换内容到引入的文件 	<head th:include="layout :: htmlhead" th:with="title='xx'"></head> />
th:replace 	布局标签，替换整个标签到引入的文件 	<div th:replace="fragments/header :: title"></div>
th:selected 	selected选择框 选中 	th:selected="(${xxx.id} == ${configObj.dd})"
th:src 	图片类地址引入 	<img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" />
th:inline 	定义js脚本可以使用变量 	<script type="text/javascript" th:inline="javascript">
th:action 	表单提交的地址 	<form action="subscribe.html" th:action="@{/subscribe}">
th:remove 	删除某个属性 	<tr th:remove="all"> 1.all:删除包含标签和所有的孩子。
th:attr 	设置标签属性，多个属性可以用逗号分隔 	比如 th:attr="src=@{/image/aa.jpg},title=#{logo}"，此标签不太优雅，一般用的比较少。
```

### 4、试图解析器源码分析怎么走的大致意思

首先 来到 ContentNegotiatingViewResolver 类中  该类实现了ViewResolver所以他也算一个视图解析器类 来到resolveViewName方法

```java
@Override
@Nullable
public View resolveViewName(String viewName, Locale locale) throws Exception {
   RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
   Assert.state(attrs instanceof ServletRequestAttributes, "No current ServletRequestAttributes");
   List<MediaType> requestedMediaTypes = getMediaTypes(((ServletRequestAttributes) attrs).getRequest());
   if (requestedMediaTypes != null) {
       //getCandidateViews这个方法获取候选的试图 里面循环了一下子  下面有
      List<View> candidateViews = getCandidateViews(viewName, locale, requestedMediaTypes);
       //getBestView 得到最好的视图  Best最好的意思
      View bestView = getBestView(candidateViews, requestedMediaTypes, attrs);
      if (bestView != null) {
         return bestView;
      }
   }

   String mediaTypeInfo = logger.isDebugEnabled() && requestedMediaTypes != null ?
         " given " + requestedMediaTypes.toString() : "";

   if (this.useNotAcceptableStatusCode) {
      if (logger.isDebugEnabled()) {
         logger.debug("Using 406 NOT_ACCEPTABLE" + mediaTypeInfo);
      }
      return NOT_ACCEPTABLE_VIEW;
   }
   else {
      logger.debug("View remains unresolved" + mediaTypeInfo);
      return null;
   }
}

//获取候选的视图  方法
private List<View> getCandidateViews(String viewName, Locale locale, List<MediaType> requestedMediaTypes)
			throws Exception {

		List<View> candidateViews = new ArrayList<>();
		if (this.viewResolvers != null) {
			Assert.state(this.contentNegotiationManager != null, "No ContentNegotiationManager set");
			for (ViewResolver viewResolver : this.viewResolvers) {
				View view = viewResolver.resolveViewName(viewName, locale);
				if (view != null) {
					candidateViews.add(view);
				}
				for (MediaType requestedMediaType : requestedMediaTypes) {
					List<String> extensions = this.contentNegotiationManager.resolveFileExtensions(requestedMediaType);
					for (String extension : extensions) {
						String viewNameWithExtension = viewName + '.' + extension;
						view = viewResolver.resolveViewName(viewNameWithExtension, locale);
						if (view != null) {
							candidateViews.add(view);
						}
					}
				}
			}
		}
		if (!CollectionUtils.isEmpty(this.defaultViews)) {
			candidateViews.addAll(this.defaultViews);
		}
		return candidateViews;
	}
```

写一个试图解析器

```java
@Configuration//跟SpringBoot说 现在MyMvcConfig 类是一个配置类了
public class MyMvcConfig implements WebMvcConfigurer {
    //写一个试图解析器
    
    //注册到ioc容器里面
    @Bean
    public ViewResolver MyisViewResolver(){
        return new MyViewResolver();
    }
    //实现ViewResolver 试图解析器接口
    public static class MyViewResolver implements ViewResolver{
        @Override
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            return null;
        }
    }
}
```

测试

   首先来到DispatcherServlet 类里面找到doDispatch 方法打个断点  Debug一下可以找到刚注册的试图解析器   结果如下

![](E:\SpringBoot笔记\images\e.PNG)

###  5、crud （增删改查细节操作）

只要没改java代码 可以不重启服务器  但是要 清理thymeleaf 缓存在 properties中  spring.thymeleaf.cache=true  在按ctrl+F9 就可以了

heml操作

```html
@{/user/login}修饰的话路径改变会自动补齐 引用资源可以用  
<form name="myForm" class="form-signin" th:action="@{/user/login}" ng-submit="save()" ng- 
      controller="loginController" method="post">  请求方式为post
    <h1 class="form-signin-heading">请登录</h1>
    <input type="text" class="input-block-level" th:name="username" placeholder="userName" ng-model="userName" required name="userName">
    <input type="password" class="input-block-level" th:name="password" placeholder="Password" ng-model="passWord" required name="passWord">
    //重点  之前一直在想弹框怎么做 现在 直接给提示就行 后台判断以后这里判断后台传来的数据是不是非空的是则输出后台传来的数据
    th:if="${not #strings.isEmpty(msg)}用来判断not 取反  th:text="${msg}"标签输出     style="color : red" css样式
    <p style="color : red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
    <label class="checkbox">
        <input type="checkbox" value="remember-me"> 下次自动登录
    </label>
    <button class="btn btn-large btn-primary" type="submit" >登录</button>
    <!--<button class="btn btn-large btn-primary" type="submit" ng-disabled="myForm.$invalid" >登录</button>-->
</form>
```



controller

```java
这里用了Thymeleaf数据引擎
public class LoginController {
    @RequestMapping(value = "/user/login" ,method = RequestMethod.POST)
    public String loginuser(Map<String,Object> map,
                            @RequestParam("username") String username,
                            @RequestParam("password") String password){
        System.out.println(username+" "+password);
        if (!username.isEmpty()&& "123456".equals(password)){
            return "index";
        }
        else {
            map.put("msg","账号密码错误");
            return "login";
        }
    }
```

默认进登录页

```java
@Configuration
public class MyConfig implements WebMvcConfigurer {   继承WebMvcConfigurer 重写addViewControllers 
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/login.html").setViewName("login");
    }
}
```

### 6、拦截器

拦截器具体代码

```java
public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取session中的数据
        Object user = request.getSession().getAttribute("loginUser");
        if (user==null){//判断session有没有数据  有登录成功  没有拦截下来转发到登录页面
            request.setAttribute("msg","没有权限等不进去");
//            response.sendRedirect("/");
            request.getRequestDispatcher("/").forward(request,response);//转发到注册拦截器的位置  在下面
            return false;
        }else{
            return true;
        }
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }
}
注册拦截器代码
@Configuration
public class MyConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("login");  转发到这里  这里在转发到登录页
        registry.addViewController("/login.html").setViewName("login");
        registry.addViewController("/index.html").setViewName("index");
    }
//注册拦截器代码  静态资源也需要放行 重点  硅谷的老师说可以不用放行害的我搞了半天
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyIn()).addPathPatterns("/**").
                excludePathPatterns("/","/login.html", "/user/login","/css/**","/js/**","/img/**");
    }
}
Controller 层代码
    
@Controller
public class LoginController {
    @RequestMapping(value = "/user/login" ,method = RequestMethod.POST)
    public String loginuser(Map<String,Object> map,
                            @RequestParam("username") String username,
                            @RequestParam("password") String password, HttpSession session){
        System.out.println(username+" "+password);
        if (!username.isEmpty()&& "123456".equals(password)){
            //登录后将账号数据放到session中便于判断是否登录过
            session.setAttribute("loginUser",username);
            return "redirect:/index.html";
        }
        else {
            map.put("msg","账号密码错误");
            return "login";
        }
    }
}
```

### 7、好用小技巧

```
th:fragment="navbar" 将html页面模块化 换个地方用可以直接引用 
<div th:insert="~{index::navbar}"></div>  引用方式  index 页面的名称 :: 引用的名字 th:fragment="navbar"这个名
日期格式化
${#dates.format(属性,'yyyy-mm-dd HH:mm:ss')}属性 如 xxx.xx
```



### 8、SpringBoot连接 jdbc

1.各种依赖

```xml
  <!--druid -->   德鲁伊
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.4</version>
        </dependency>
        <!-- log4j -->  
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!--        web依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
<!--        jdbc依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
<!--mysql依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```



2.yaml 配置

```properties
spring:
  datasource:
    #   数据源基本配置
    url: jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
```

源码分析

依旧是根据porperties配置来分析

DataSourceProperties中可以配置已有的属性  账号密码等 具体自己去看

```java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {
    private String username;
	private String password;
}

```

test测试

```java
	@Autowired
    DataSource dataSource; 将DataSource添加到ioc容器中
    @Test
    void contextLoads() throws SQLException {
        System.out.println(dataSource.getClass()); //获取数据源 数据库连接池  
        //默认是class com.zaxxer.hikari.HikariDataSource
  
        Connection connection = dataSource.getConnection(); //获取到 driver-class-name: com.mysql.cj.jdbc.Driver
        System.out.println(connection);
        connection.close();
    }
```

### 9、druid 德鲁伊

依赖上面已经导了

```xml
	<!--druid -->   德鲁伊
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.4</version>
        </dependency>
```

优点 

可以监控数据库的一举一动 需要配置

来吧展示

properties   

```properties
spring:
  datasource:
    #   数据源基本配置
    url: jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource   修改数据源
    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    schema:  执行sql语句
      - calsspath:路径 sql路径
      - calsspath:路径 sql路径
```

虽然我们配置了druid连接池的其它属性，但是不会生效。因为默认是使用的java.sql.Datasource的类来获取属性的，有些属性datasource没有。如果我们想让配置生效，需要手动创建Druid的配置文件

自创一个config如图

![](E:\SpringBoot笔记\images\f.PNG)

```JAVA
@ConfigurationProperties(prefix = "spring.datasource")  找到配置文件
@Bean
public DruidDataSource druidDataSource(){
      return new DruidDataSource();
}
```

**Druid的最强大之处在于它有着强大的监控，可以监控我们发送到数据库的所有sql语句。方便我们后期排插错误。**

我们接着在DruidDataSource里面配置监控中心：

```JAVA
/**
     * 配置监控服务器
     * @return 返回监控注册的servlet对象
     * @author SimpleWu
     固定写法粘上去就行   还有一些属性可以看源码 StatViewServlet 和他的父类  父类中比较实用 
     FilterRegistrationBean 也一样
     */
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        // 添加IP白名单
        servletRegistrationBean.addInitParameter("allow", "127.0.0.1");
        // 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高
        servletRegistrationBean.addInitParameter("deny", "127.0.0.1");
        // 添加控制台管理用户
        servletRegistrationBean.addInitParameter("loginUsername", "SimpleWu");
        servletRegistrationBean.addInitParameter("loginPassword", "123456");
        // 是否能够重置数据
        servletRegistrationBean.addInitParameter("resetEnable", "false");
        return servletRegistrationBean;
    }

    /**
     * 配置服务过滤器
     *
     * @return 返回过滤器配置对象
     */
    @Bean
    public FilterRegistrationBean statFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        // 添加过滤规则
        filterRegistrationBean.addUrlPatterns("/*");
        // 忽略过滤格式
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*,");
        return filterRegistrationBean;
    }
```

项目启动后访问路径 http://localhost:8080/druid  可以看到项目的一举一动 如图

![](E:\SpringBoot笔记\images\g.PNG)

### 10、SpringBoot 整合Mybatis

不是SpringBoot自带的 是Mybatis官方自己写的

```xml
<!-- mybatis-spring-boot-starter 整合包-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```

pojo 不多说

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
private int id;
private String name;
private String pwd;
}
```

Mapper接口

```java
@Mapper  //让springBoot知道有Mapper这个东西存在  也可以在启动项哪里写一个@MapperScan(路径)
@Repository  //将他注入到springioc中
public interface MapperUser {
    List<User> queryList();
}
这样也行能找到
@SpringBootApplication
@MapperScan(路径)
public class SpringBootMybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMybatisApplication.class, args);
    }

}

```

mapper配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lsm.mapper.MapperUser">
    <select id="queryList" resultType="User">
        select * from user
    </select>
</mapper>
```

重点是yaml配置文件配置

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
    
整合mybatis 
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml  找到xml配置文件的路径
  type-aliases-package: com.lsm.pojo  //取别名
```

测试

```java
@Test
void d() throws SQLException {
    List<User> users = mapperUser.queryList();
    System.out.println(users);
}

controller测试  随便玩
    
@Controller
public class hellocontroller {
    @Autowired
    private MapperUser mapperUser;
    @ResponseBody
    @RequestMapping("select")
    public List<User> select(){
        List<User> users = mapperUser.queryList();
        for (User user : users) {
            System.out.println(user);
        }
        return users;
    }
}
```

### 11、springsecurity 安全框架个什么鬼的东西

![](E:\SpringBoot笔记\images\h.PNG)

​	写一个类继承 WebSecurityConfigurerAdapter

​		依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
  <!-- 整合thymeleaf security -->
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity4</artifactId>
            <version>3.0.4.RELEASE</version>
        </dependency>
```

​	写一个类	

@EnableWebSecurity要写这个注释

```java
@EnableWebSecurity
public class pringSecurity extends WebSecurityConfigurerAdapter {
    //    授权
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //首页所有人可以访问，功能也只有对应有权限的人才能访问
        //请求授权的规则
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/log1/**").hasRole("log1")
                .antMatchers("/log2/**").hasRole("log2")
                .antMatchers("/log3/**").hasRole("log3");
        //没有权限进入登录页面
        http.formLogin();
        //注销功能
        http.logout().logoutSuccessUrl("/");
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("root").password(new BCryptPasswordEncoder().encode("123")).roles("log1", "log2", "log3").and()
                .withUser("admin").password(new BCryptPasswordEncoder().encode("123")).roles("log2", "log3").and()
                .withUser("youke").password(new BCryptPasswordEncoder().encode("123")).roles("log3");
    }
}
```



heml页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" //thymeleaf注释 可以给出有提示
      //thymeleaf整合springsecurity 注释给提示
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">  
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div sec:authorize="!isAuthenticated()">  //判断登录没 没登陆不显示
    <a th:href="@{login}">登录</a>
</div>
<div sec:authorize="isAuthenticated()">   //登录后显示
    角色:<p sec:authentication="name"></p>  显示账号
 <a th:href="@{logout}">注销</a><br>
</div>
<a th:href="@{log1}">用户一</a><br>
<a th:href="@{log2}">用户二</a><br>
<a th:href="@{log3}">用户三</a>
</body>
</html>
```



### 12、Shiro安全框架

![](E:\SpringBoot笔记\images\i.PNG)

​	pom.xml架包

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.7.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

​		config配置文件 配置三个必须@Bean

```java
@Configuration
public class Myshiro {
    @Bean
    public ShiroFilterFactoryBean getshiroFilterFactoryBean(@Qualifier("getdefaultWebSecurityManager") DefaultWebSecurityManager getdefaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(getdefaultWebSecurityManager);
        //登录拦截
        Map map=new LinkedMap();
        map.put("/user/update","authc");
        map.put("/user/add","authc");
        bean.setFilterChainDefinitionMap(map);
        //开启登录验证
        bean.setLoginUrl("/login");
        return bean;
    }

    @Bean
    public DefaultWebSecurityManager getdefaultWebSecurityManager(@Qualifier("getuserRealm") UserRealm getuserRealm){
        DefaultWebSecurityManager bean = new DefaultWebSecurityManager();
        bean.setRealm(getuserRealm);
        return bean;
    }
    @Bean
    public UserRealm getuserRealm(){
       return new UserRealm();  这个东西是下面那个类
    }
}
```



```java
public class UserRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        System.out.println("也来过doGetAuthorizationInfo");
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println("也来过doGetAuthorizationInfo");
        String name="root";
        String pwd="123";
        UsernamePasswordToken token1 = (UsernamePasswordToken) token;
        System.out.println(token1.getUsername());
        if (!token1.getUsername().equals(name)){
            return null;
        }
        return new SimpleAuthenticationInfo("",pwd,"");
    }
}
```

```java

controller
@Controller
public class hellocontroller {
    @RequestMapping({"/","/index"})
    public String index(Model model){
        model.addAttribute("msg","我想搞事");
        return "index";
    }


    @RequestMapping("/user/add")
    public String add(Model model){
        model.addAttribute("msg","我想搞事");
        return "vue/add";
    }
    @RequestMapping("/user/update")
    public String update(Model model){
        model.addAttribute("msg","我想搞事");
        return "vue/update";
    }


    @RequestMapping("/login")
    public String login(Model model){
        model.addAttribute("msg","我想搞事");
        return "login";
    }
 
    @RequestMapping("/tologin") 重点
    public String tologin(String username, String pwd, Model model){
        Subject currentUser = SecurityUtils.getSubject();  看不懂的都是固定代码
        UsernamePasswordToken token = new UsernamePasswordToken(username, pwd); 令牌 账号密码通过RequestMapping 前端页面传过来
        try {
            currentUser.login(token);   如果没异常就能登录成功
            return "index";
        } catch (UnknownAccountException uae) {  账号错误异常
            model.addAttribute("msg","账号错误");
            return "login";
        } catch (IncorrectCredentialsException ice) { 密码错误异常
            model.addAttribute("msg","密码错误");
            return "login";
        }

    }
}
```

### 13、shiro整合mybatis

pom.xml

```xml
<dependencies>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.16</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.7.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

​	写一下数据源算了 太简单了 而且又多

```yaml
spring:
  datasource:
    #   数据源基本配置
    url: jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    type: com.alibaba.druid.pool.DruidDataSource
    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
#整合mybatis  就这一句话 
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

​	springBoot 整合mybatis成功后调用一下 查询的方法吧之前固定的账号密码换一下就好了

```java
@Autowired
serverUser serverUser;
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    System.out.println("也来过doGetAuthorizationInfo");
    return null;
}

@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    System.out.println("也来过doGetAuthorizationInfo");
    UsernamePasswordToken token1 = (UsernamePasswordToken) token;
    User user = serverUser.queryByName(token1.getUsername());  //查询出单个信息
    if (user==null){ 判断账号是否为空  空就返回 UnknownAccountException 账号错误异常 null就是账号错误异常
        return null; 
    }
    return new SimpleAuthenticationInfo("",user.getPwd(),"");  判断密码是否正确 这里密码底层进行了加密  
}
```



权限设置

登录拦截修改

```
@Configuration
public class MyShiro {
    //ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getshiroFilterFactoryBean(@Qualifier("getdefaultWebSecurityManager") DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
//       设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);
    /*
        anon :无需认证就可以访问
        authc: 必须认证了才能访问
        user :必须拥有记住我功能才能用
        perms： 拥有对某个组员的权限才能访问
        role:拥有某个校色权限才能访问
     */
//        登录拦截
        Map map = new LinkedHashMap();

        map.put("/user/add", "perms[user:add]");//设置权限perms拥有对某个组员的权限才能访问
        map.put("/user/update", "perms[user:update]");  
                           //[user:update]user:update通过这个字符串进行权限标识
        map.put("/user/*", "authc");
        bean.setFilterChainDefinitionMap(map);
        //未经授权要调到指定页面
        bean.setUnauthorizedUrl("/unauthorized");
        //跳转到登录页面
        bean.setLoginUrl("/login");

        return bean;
    }

    //DafaultWebSecurityManager
    @Bean
    public DefaultWebSecurityManager getdefaultWebSecurityManager(@Qualifier("getuserRealm") UserRealm getuserRealm) {
        DefaultWebSecurityManager bean = new DefaultWebSecurityManager();
        bean.setRealm(getuserRealm);
        return bean;
    }

    //创建realm对象，需要自定义类
    @Bean
    public UserRealm getuserRealm() {
        return new UserRealm();
    }
}
```

将查到的数据通过Subject currentUser = SecurityUtils.getSubject() 进行传递

 Subject currentUser = SecurityUtils.getSubject();
        User user = (User) currentUser.getPrincipal(); 将权限拿出来
        info.addStringPermission(user.getQuanxain()); 放到这里给他指定数据库中的权限

addStringPermission 放心权限

```java
public class UserRealm extends AuthorizingRealm {
    @Autowired
    serverUser serverUser;
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行了授权 ==> doGetAuthorizationInfo");
        SimpleAuthorizationInfo info=new SimpleAuthorizationInfo();
//        info.addStringPermission("user:add");//给权限进去 缺点固定死了不灵活
        Subject currentUser = SecurityUtils.getSubject();
        User user = (User) currentUser.getPrincipal();
        System.out.println(user.getQuanxain());
        info.addStringPermission(user.getQuanxain());
        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println("执行了授权 ==> doGetAuthorizationInfo");

        UsernamePasswordToken token1 = (UsernamePasswordToken) token;
        User user = serverUser.queryByName(token1.getUsername());
        if (user==null){
            return null;
        }
        将查到的数据通过Subject currentUser = SecurityUtils.getSubject() 进行传递
        return new SimpleAuthenticationInfo(user,user.getPwd(), "");
    }
}
```

### 14、shiro整合 thymeleaf

​	jar包

```xml
<!-- thymeleaf-extras-shiro  thymeleaf 整合shiro-->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
配置文件
@Configuration
public class MyShiro {
    //ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getshiroFilterFactoryBean(@Qualifier("getdefaultWebSecurityManager") DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
//       设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);
    /*
        anon :无需认证就可以访问
        authc: 必须认证了才能访问
        user :必须拥有记住我功能才能用
        perms： 拥有对某个组员的权限才能访问
        role:拥有某个校色权限才能访问
     */
//        登录拦截
        Map map = new LinkedHashMap();

        map.put("/user/add", "perms[user:add]");//设置权限perms拥有对某个组员的权限才能访问
        map.put("/user/update", "perms[user:update]");
        map.put("/user/*", "authc");
        bean.setFilterChainDefinitionMap(map);
        //未经授权要调到指定页面
        bean.setUnauthorizedUrl("/unauthorized");
        //跳转到登录页面
        bean.setLoginUrl("/login");

        return bean;
    }

    //DafaultWebSecurityManager
    @Bean
    public DefaultWebSecurityManager getdefaultWebSecurityManager(@Qualifier("getuserRealm") UserRealm getuserRealm) {
        DefaultWebSecurityManager bean = new DefaultWebSecurityManager();
        bean.setRealm(getuserRealm);
        return bean;
    }

    //创建realm对象，需要自定义类
    @Bean
    public UserRealm getuserRealm() {
        return new UserRealm();
    }
	加这一个就行了
    @Bean
    public ShiroDialect getShiroDialect(){
        return new ShiroDialect();
    }
}

```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://www.thymeleaf.org/thymeleaf-extras-shiro"> 
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<h1 th:text="${msg}"></h1>
<!--/*@thymesVar id="getAttribute" type="at"*/-->
<div th:if="${session.loginin==null}">  
    <a th:href="@{login}">登录</a>
</div>


<div shiro:hasPermission="user:add">  整合出来能用的标签
    <a th:href="@{/user/add}">添加</a>
</div>
<div shiro:hasPermission="user:update">
    <a th:href="@{/user/update}">修改</a>
</div>
</body>
</html>
```



### 15、Swagger使用

#### Swagger 的优势

- 支持 API 自动生成同步的在线文档：使用 Swagger 后可以直接通过代码生成文档，不再需要自己手动编写接口文档了，对程序员来说非常方便，可以节约写文档的时间去学习新技术。
- 提供 Web 页面在线测试 API：光有文档还不够，Swagger 生成的文档还支持在线测试。参数和格式都定好了，直接在界面上输入参数对应的值即可在线测试接口

jar包

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>   3.0.0版本的访问不到 http://localhost:8081/swagger-ui.html 文档
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>3.0.0版本的访问不到 http://localhost:8081/swagger-ui.html 文档
</dependency>
```

写一个类配置swagger   什么都不写就全部默认 写两个注解就行了   这里只是改了一下默认配置

有一些@Api的注解 就是给在线文档上加注释的 没深究

```java
@Configuration
@EnableSwagger2
public class Swagger {
    @Bean
    public Docket a(Environment environment){
        return new Docket(DocumentationType.SWAGGER_2).groupName("A");
    }

    @Bean
    public Docket b(Environment environment){
        return new Docket(DocumentationType.SWAGGER_2).groupName("B");
    }

    //设置要显示的Swagger开发环境
    //配置Swagger的Docket的bean实例
    @Bean
    public Docket docket(Environment environment){
        //设置要显示的Swagger开发环境
        Profiles dev = Profiles.of("dev");
        System.out.println(dev);
        //通过environment.acceptsProfiles判断是否处在租户设定的环境中
        boolean b = environment.acceptsProfiles(dev);
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo())
                .enable(b) //  是否启动swagger  false 就不启动
                .groupName("龟儿子") //起名字
                .select()
//                RequestHandlerSelectors  配置要扫描接口的方式
//                withClassAnnotation 扫描类上的注解
//                withMethodAnnotation 扫描方法上的注解
//                any  全部扫描
//                none  全部不扫描
//                basePackage 扫描指定包
                .apis(RequestHandlerSelectors.basePackage("com.lsm.controller"))
//                paths() 过滤什么路径
                .paths(PathSelectors.ant("lsm/**"))
                .build();
    }
    public ApiInfo apiInfo(){
        Contact contact = new Contact("didi", "localhost:8080", "825784179@qq.com");
       return new ApiInfo("龟儿子",
               "嘿嘿",
               "v1.0",
               "龟儿子:tos",
               contact,
               "Apache 2.0",
               "http://www.apache.org/licenses/LICENSE-2.0",
               new ArrayList<VendorExtension>());
    }
}
```

###   16、异步任务@Async

开启一个一步任务 跟多线程一样 开启了一个线程完成某件事 两个线程去完成  用法也很简单 在想用的地方加一个注释@Async（告诉spring这是一个异步方法）  在启动雷上加一个@EnableAsync  （这个叫开启异步注解功能）

案例如下



controller

```java
@RestController
public class hello {
    @Autowired
    user user;
    @RequestMapping("/hello")
    public String aa(){
    user.aa();
    return "搞定了";
    }
}
```

service

```java
@Service
public class user {
    @Async //告诉spring这是一个异步方法
    public void aa(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("程序正在处理");
    }
}
```

启动项具体代码

```java
@EnableAsync //开启异步注解功能
@SpringBootApplication
public class SpringbootAsyncApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAsyncApplication.class, args);
    }

}
```