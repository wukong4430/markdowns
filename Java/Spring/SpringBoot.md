# 1 微服务

微服务是一种架构风格。 力求：**高内聚，低耦合。**

## 1.1 对比最原始的架构

- all in one:

    这种架构方式，我们把所有的e





## 1.2 特性

1. “组件化”， “多服务”

    - 组件是一个可以独立更换和升级的软件单元。
    - 服务可以单独部署，如果一个应用系统**[4]**由在单个进程中的多个软件库所组成，那么对任一组件做一处修改，都不得不重新部署整个应用系统。
    - 使用服务部署，比起进程内调用，远程调用更加昂贵。

2. **围绕“业务功能”组织团队**

    

    1. 按照智能分：容易产生孤岛式![image-20200627162830027](SpringBoot.assets/image-20200627162830027.png)
    2. 按照业务功能（business capability)划分，![image-20200627163018185](SpringBoot.assets/image-20200627163018185.png)

3. **“做产品”而不是“做项目”**

4. **“智能端点”与“傻瓜管道”** （smart endpoint and dumb pipes)

5. 









# 2 SpringBoot 原理



自动配置：



**pom.xml**

- spring-boot-dependencies: 核心依赖在父工程中。
- 我们在写或者引入SpringBoot依赖的时候，不需要指定版本。因为有仓库。



## **2.1 启动器**



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

就是SpringBoot的启动场景。

- 比如spring-boot-starter-web， 他会帮助我们自动导入web应用所需要的环境依赖。
- springboot会将所有的功能场景，都变成一个个的启动器
- 我们需要使用什么功能，就只需要找到对应的启动器就可以了。



> ```yaml
> # LazyInitialization
> spring：
> 	lazy-initialization: true
> ```





## **2 .2 主程序**



```java
//@SpringBootApplication ： 标注这个类是一个springboot的应用
// same as @Configuration @EnableAutoConfiguration @ComponentScan
@SpringBootApplication 
public class Springboot01HelloApplication {

    public static void main(String[] args) {
        // 将springboot应用启动
        SpringApplication.run(Springboot01HelloApplication.class, args);
    }

}
```



## **2.3 注解：**





> ## @SpringBootApplication



作用：标注在某个类上说明这个类是SpringBoot的主配置类 ， SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

进入这个注解：可以看到上面还有很多其他注解！

 ```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // ......
}
 ```



> A single `@SpringBootApplication` annotation can be used to enable those three features, that is:
>
> - `@EnableAutoConfiguration`: enable [Spring Boot’s auto-configuration mechanism](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/using-spring-boot.html#using-boot-auto-configuration)
> - `@ComponentScan`: enable `@Component` scan on the package where the application is located (see [the best practices](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code))
> - `@Configuration`: allow to register extra beans in the context or import additional configuration classes



如果不想使用@SpringBootApplication注解 （一次性使用三个功能太多了）， 那么可以分开使用：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {

    public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
    }

}
```

这里，就不直接使用@ComponentScan注解，而是换成了 @Import， 来显式的导入我们需要的组件！



> ## @ComponentScan

进入SpringBootApplication可以看到ComponentScan这个注解。它对应了XML配置中的元素。

作用：自动扫描并加载符合条件的组件或者bean ， 将这个bean定义加载到IOC容器中。**Scan**

这个注解再往下走，没有包含其他特殊的类了。



> ## @SpringBootConfiguration

@SpringBootApplication中另一个非常重要的注解。

作用：SpringBoot的配置类 ，标注在某个类上 ， 表示这是一个SpringBoot的配置类；

```java
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

再往上一级就是@Configuration，而@Configuration就熟悉了。再往上一级是@Component。也就是我们熟悉的Spring组件!

> 这里的 @Configuration，说明这是一个配置类 ，配置类就是对应Spring的xml 配置文件；
>
> 里面的 @Component 这就说明，启动类本身也是Spring中的一个组件而已，负责启动应用！







> ## @EnableAutoConfiguration

这个注解负责：**开启自动配置功能**。

以前我们需要自己配置的东西，而现在SpringBoot可以自动帮我们配置 ；@EnableAutoConfiguration 告诉SpringBoot开启自动配置功能，这样自动配置才能生效；

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

   /**
    * Exclude specific auto-configuration classes such that they will never be applied.
    * @return the classes to exclude
    */
   Class<?>[] exclude() default {};

   /**
    * Exclude specific auto-configuration class names such that they will never be
    * applied.
    * @return the class names to exclude
    * @since 1.3.0
    */
   String[] excludeName() default {};

}
```





> ## @AutoConfigurationPackage：自动配置包



```Java
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```

@Import: Spring底层注解。给容器中导入一个组件；

AutoConfigurationPackages.Registrar.class： 将主启动类的所在包以及包下面所有子包里面的组件扫描的Spring容器中；



> @Import(AutoConfigurationImportSelector.class) : 给容器导入组件；



AutoConfigurationImportSelector：自动配置导入选择器，那么它会导入哪些组件的选择器呢？我们点击去这个类看源码：

1、这个类中有一个这样的方法

 getCandidateConfigurations

```java
/** 获得候选的配置
*/
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
         getBeanClassLoader());
   Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
         + "are using a custom packaging, make sure that file is correct.");
   return configurations;
}
```

2、这个方法又调用了  SpringFactoriesLoader 类的静态方法！我们进入SpringFactoriesLoader类loadFactoryNames() 方法



再往下追溯，可以发现

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    String factoryTypeName = factoryType.getName();
    return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
}

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
...
}
```

> 在loadSpringFactories方法中，多次出现了一个文件：**"META-INF/spring.factories"**,这个就是我们需要的spring配置工厂。

![image-20200628141434301](SpringBoot.assets/image-20200628141434301.png)





## 2.4 spring.factories 配置

在这个文件中，保存了大量的AutoConfiguration类。这些都是JavaConfig类。（代替Spring中的xml）

随便打开一个查看源码：**WebMvcAutoConfiguration**

![image-20200628141723281](SpringBoot.assets/image-20200628141723281.png)

所以，

- 自动配置真正实现是从classpath中搜寻所有的META-INF/spring.factories配置文件 ，
- 并将其中对应的 org.springframework.boot.autoconfigure. 包下的配置项，通过反射实例化为对应标注了 @Configuration的JavaConfig形式的IOC容器配置类 ， 
- 然后将这些都汇总成为一个实例并加载到IOC容器中。

**需要注意的是：**

spirng.factories中的这么多配置并不会全部生效，因为有注解@ConditionalOnClass，只要在条件合适的情况下**(导入了对应的starter）**才能生效!



## 2.5 Run 方法

>  SpringApplication

**这个类主要做了以下四件事情：**(重要)

1、推断应用的类型是普通的项目还是Web项目

2、查找并加载所有可用初始化器 ， 设置到initializers属性中

3、找出所有的应用程序监听器，设置到listeners属性中

4、推断并设置main方法的定义类，找到运行的主类

![img](SpringBoot.assets/640.jpg)





# 3 Application.properties 配置

## 3.1 外部配置的优先级 （随着版本更新 可能会变化 目前是2.1.6）

1. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/using-boot-devtools.html#using-boot-devtools-globalsettings) on your home directory (`~/.spring-boot-devtools.properties` when devtools is active).
2. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.1.8.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
3. `properties` attribute on your tests. Available on [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html) and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests).
4. Command line arguments.
5. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
6. `ServletConfig` init parameters.
7. `ServletContext` init parameters.
8. JNDI attributes from `java:comp/env`.
9. Java System properties (`System.getProperties()`).
10. OS environment variables.
11. A `RandomValuePropertySource` that has properties only in `random.*`.
12. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties) outside of your packaged jar (`application-{profile}.properties` and YAML variants).
13. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties) packaged inside your jar (`application-{profile}.properties` and YAML variants).
14. Application properties outside of your packaged jar (`application.properties` and YAML variants).
15. Application properties packaged inside your jar (`application.properties` and YAML variants).
16. [`@PropertySource`](https://docs.spring.io/spring/docs/5.1.8.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes.
17. Default properties (specified by setting `SpringApplication.setDefaultProperties`).



## 3.2 application.yaml 配置 （建议）

POJO:

```java
public class Person {

    private String name;
    private List<String> hobbys;
    private int age;
    private String sex;
    private Dog dog;
    private String location;
    private Map<String, Object> maps;
```

```java
public class Dog {

    private String name;
    private int age;
```



application.yaml

```yaml
person:
  name: Kaige
  hobbys:
    - code
    - soccer
    - swim
  age: 23
  sex: male
  location: NB
  maps: {k1: v1, k2: v2}
  dog:
    name: 小黑
    age: 3
```

 

Then 添加注解

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private List<String> hobbys;
    private int age;
    private String sex;
    private Dog dog;
    private String location;
    private Map<String, Object> maps;
```

这样就能直接注入属性了！



同时，yaml还支持需要 ${} 占位符操作。比如 ${random.int} ${random.uuid}。可以在配置文件中完成这部分代码，就可以减少不必要的代码。



```yaml
debug: true # 用于查看哪些类生效，哪些不生效！
```







## 3.3 与spring.factories的联系

首先，spring.factories下的XxxAutoConfiguration类都有注解 @Configuration。表示这个是个配置类！

举例，![image-20200629141403138](SpringBoot.assets/image-20200629141403138.png)

展开

![image-20200629141417065](SpringBoot.assets/image-20200629141417065.png)

- @Configuration：表示这是一个配置类
- @EnableConfigurationProperties：
    - ServierProperties：![image-20200629141658648](SpringBoot.assets/image-20200629141658648.png)
    - 带有一个@ConfigurationProperties注解，表明能从.yaml中取配置。字段：port、address等就是application.yaml中可以配置的属性！
    - ![image-20200629141922621](SpringBoot.assets/image-20200629141922621.png)
- @ContionalOnWebApplication：Spring的底层注解：根据不同的条件将某个Bean加载到应用上下文中。







## 3.4 ConditionalOnXxx



| @ConditionalOnProperty          | application.properties 或 application.yml 文件中 mybean.enable 为 true 才会加载 MyCondition 这个 Bean，如果没有匹配上也会加载，因为 matchIfMissing = true，默认值是 false。 |
| ------------------------------- | ------------------------------------------------------------ |
| **@ConditionalOnBean**          | 某个（另外的）Bean存在时加载                                 |
| **ConditionalOnMissingBean**    | 某个（另外的）Bean不存在时加载                               |
| @ConditionalOnClass             | 某个（另外的）类存在与classpath中加载                        |
| @ConditionalOnMissingClass      | 某个（另外的）类不存在与classpath中加载                      |
| @ConditionalOnExpression        | 多个复杂属性判断                                             |
| @ConditionalOnSingleCandidate   |                                                              |
| @ConditionalOnResource          | bean依赖的资源存在，比如logback.xml                          |
| @ConditionalOnJndi              | 只有指定的资源通过 JNDI 加载后才加载 bean                    |
| @ConditionalOnJava              | 只有运行指定版本的 Java 才会加载 Bean                        |
| @ConditionalOnWebApplication    | 只有运行在 web 应用里才会加载这个 bean                       |
| @ConditionalOnNotWebApplication |                                                              |
| @ConditionalOnCloudPlatform     | 特定的云平台下运行                                           |



## 3.5 自动装配的原理! （重点）

1. SpringBoot启动会加载大量的自动配置类。；
2. 查看需要的功能是否有在SpringBoot中默认写好的自动配置类中；
3. 再看自动配置类中到底配置了哪些组件；（只要我们需要的组件存在其中，我们就不需要再手动配置了）
4. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们只需要再配置文件中指定这些属性的值即可；
    1. xxxAutoConfiguration：自动配置类；给容器添加组件
    2. xxxProperties：封装配置文件中的相关属性；
    3. 2 和 **application.yaml**的属性值相互对应。







# 4 JRS 303 校验

在Java类上添加注解 @Validated； 在需要校验的字段上添加 注解。

```java
@Component //注册bean
@ConfigurationProperties(prefix = "person")
@Validated  //数据校验
public class Person {
    @Email(message="邮箱格式错误") //name必须是邮箱格式    
    private String name;}
```



## 4.1 常用参数

```yaml

@NotNull(message="名字不能为空")
private String userName;
@Max(value=120,message="年龄最大不能查过120")
private int age;
@Email(message="邮箱格式错误")
private String email;

空检查
@Null       验证对象是否为null
@NotNull    验证对象是否不为null, 无法查检长度为1的字符串
@NotBlank   检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
@NotEmpty   检查约束元素是否为NULL或者是EMPTY.
    
Booelan检查
@AssertTrue     验证 Boolean 对象是否为 true  
@AssertFalse    验证 Boolean 对象是否为 false  
    
长度检查
@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  
@Length(min=, max=) string is between min and max included.

日期检查
@Past       验证 Date 和 Calendar 对象是否在当前时间之前  
@Future     验证 Date 和 Calendar 对象是否在当前时间之后  
@Pattern    验证 String 对象是否符合正则表达式的规则
```





# 5 多环境切换

同一个application.yaml可以被放在多个不同的位置：

```xml
file:./config/   		根目录下的config文件夹  				优先级1
file:./			 		根目录									优先级2
classpath:/config/		resources 下的config目录				优先级3
classpath:/				resources目录							优先级4
```

环境的切换可以通过优先级高覆盖的方式进行。



```yaml
# 默认配置
server:
  port: 8081
spring:
# 选择 哪个环境
  profiles:
    active: dev

# 用---分割
---
server:
  port: 8082
spring:
  profiles: dev
---
server:
  port: 8083

spring:
  profiles: test
```



# 6 SpringBoot Web开发



## 6.1静态资源

WebMvcAutoConfiguration.java中：

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
   if (!this.resourceProperties.isAddMappings()) {
      logger.debug("Default resource handling disabled");
      return;
   }
   Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
   CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
   if (!registry.hasMappingForPattern("/webjars/**")) {
      customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
   String staticPathPattern = this.mvcProperties.getStaticPathPattern();
   if (!registry.hasMappingForPattern(staticPathPattern)) {
      customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
            .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
            .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
   }
}
```



> 1. webjars方式

涉及到一个webjars:

打开webjars官网 https://www.webjars.org/，可以通过maven的方式导入依赖。比如导入一个jquery，我们就能在jquery的jar包下找到   

```xml
 META-INF/resources/webjars/...
```

这样，在开启服务后，我们通过访问

```
localhost:8080/webjars/jquery/0.0.0/jquery.js
```

实现静态资源的访问。



> 2. properties方式

![image-20200629154146382](SpringBoot.assets/image-20200629154146382.png)

通过getStaticLocations()：

![image-20200629154355376](SpringBoot.assets/image-20200629154355376.png)

可以看到，总共有四种路径，都支持静态资源的访问。（classpath：指的就是resources目录）

同时，这几个路径有优先级区别。 resources > static > public 。

> > 通常来说，会在public下放公共的js，static放一些图片资源，resources：upload上传的文件。



> 3. 如果在application.yaml中定义了pattern路径，那么这个优先级是最高的。前面说的webjars和四种路径都失效了！

![image-20200629154819267](SpringBoot.assets/image-20200629154819267.png)

![image-20200629155410186](SpringBoot.assets/image-20200629155410186.png)

在.yaml设置为false就无法通过 1 和 2 的方式访问静态资源了。

> > 默认的staticPathPattern = "/**"

![image-20200629155714649](SpringBoot.assets/image-20200629155714649.png)

在类

![image-20200629155735732](SpringBoot.assets/image-20200629155735732.png)

下，所以可以在yaml中配置：

![image-20200629160613223](SpringBoot.assets/image-20200629160613223.png)

现在，所有的静态资源都通过 **/heeelo** 访问。

注意：不是说静态资源放在了 classpath: heeelo下，而是必须先通过 localhost:8080/heeelo/ 来访问静态资源。依然是原本的  resources > static >  public 的优先级顺序。

> > 设置静态资源位置
> >
> > spring.resources.static-locations

修改静态资源访问路径：

![image-20200629161135782](SpringBoot.assets/image-20200629161135782.png)

现在 可以访问heeelo目录下的静态资源了！ 而且优先级比resources还要高！

> 总结

**“spring.mvc.static-path-pattern”用于阐述HTTP请求地址，而“spring.resources.static-locations”则用于描述静态资源的存放位置。**





## 6.2 首页的定制

```java
WebMvcAutoConfiguration.java:
```

![image-20200629162054648](SpringBoot.assets/image-20200629162054648.png)



index.html可以放在任意静态资源目录下。但是不能放在template下直接访问。

>  在template目录下的所有页面，只能通过controller来跳转!
>
>  同时，需要模板引擎的支持。thymeleaf







## 6.3 数据库连接

### 1 JDBC



> Spring的数据底层都是 Spring Data. 可以连接JDBC、JDA、Redis、Hadoop

创建项目时，需要勾选SQL下的工具。JDBC API + Mysql Driver



配置yaml：

```yaml
spring:
  datasource:
    username: root
    password: admin
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
    driver-class-name: com.mysql.jdbc.Driver
```

==导入了依赖和配置之后，SpringBoot就会自动创建一些Bean==。



测试一下：

```java
@SpringBootTest
class Springboot04DataApplicationTests {

    @Autowired
    DataSource dataSource;

    @Test
    void contextLoads() {
        // 查看一下默认的数据源
        System.out.println(dataSource.getClass());
        
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }

}

class com.zaxxer.hikari.HikariDataSource
// 默认用的是Hikari
```



>  在SpringBoot中有很多XXXTemplate，这些都是配置好的Bean，拿来直接用就可以。

举例：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(JdbcOperations.class)
class JdbcTemplateConfiguration {

   @Bean
   @Primary
   JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
      JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
      JdbcProperties.Template template = properties.getTemplate();
      jdbcTemplate.setFetchSize(template.getFetchSize());
      jdbcTemplate.setMaxRows(template.getMaxRows());
      if (template.getQueryTimeout() != null) {
         jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
      }
      return jdbcTemplate;
   }

}
```

![image-20200714130844647](SpringBoot.assets/image-20200714130844647.png)

在这个包下有这个一个Template，是一个bean：JdbcTemplate，我们是可以直接用的。dataSource和properties这两个参数我们都已经有了。所以直接用就可以。

**测试查询：**

```java
@RestController
public class JdbcController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    /**
     * 查询数据库的所有信息
     * @return
     */
    @RequestMapping(value = "/userList", method = RequestMethod.GET)
    public List<Map<String, Object>> userList() {
        String sql = "select * from users";
        List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql);

        return maps;
    }

}
```



### 2 Druid 最强优势: 日志的监控

> 集成了日志监控，性能也不错。

导入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.14</version>
</dependency>
```



指定数据源：

只要设置type: com.alibaba.druid.pool.DruidDataSource

```yaml
spring:
  datasource:
    username: root
    password: admin
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    
    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
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

    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```



把配置注入Spring

```java
@Configuration
public class DruidConfig {

    /**
     * 通过两个注解将yaml中的配置注入Spring 【关联起来了】
     * @return
     */
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }
}
```



开启日志监控

```java
/**
 * 后台监控：web.xml ServletRegisterationBean
 * 因为SpringBoot内置了serlvet容器，所以没有web。xml，替代ServletRegistrationBean
 * @return
 */
@Bean
public ServletRegistrationBean statViewServlet() {
    ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");

    // 后台登录需要账号密码
    HashMap<String, String> initParameters = new HashMap<>();

    // loginUsername 和 loginPassword 两个字段是固定的
    initParameters.put("loginUsername", "admin");
    initParameters.put("loginPassword", "123456..");

    // 允许谁访问 空表示所有人都可以访问
    initParameters.put("allow", "");

    // 初始化参数
    bean.setInitParameters(initParameters);
    return bean;
}
```

通过localhost:8080/druid/访问后台，==代码中必须配置这个路径。==



**添加过滤器：哪些信息不需要进行统计**

```java
//配置 Druid 监控 之  web 监控的 filter
//WebStatFilter：用于配置Web和Druid数据源之间的管理关联监控统计
@Bean
public FilterRegistrationBean webStatFilter() {
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new WebStatFilter());

    //exclusions：设置哪些请求进行过滤排除掉，从而不进行统计
    Map<String, String> initParams = new HashMap<>();
    initParams.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");
    bean.setInitParameters(initParams);

    //"/*" 表示过滤所有请求
    bean.setUrlPatterns(Arrays.asList("/*"));
    return bean;
}
```









# 7 模板引擎 Thymeleaf

导入Thymeleaf 3.x依赖：

```xml
<!--Thymeleaf 3.x。 不要用2.x-->
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>

<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
```



把html放在template下，通过controller跳转视图。



- 取值操作：

```xml
${value}
```

- 遍历

```java
model.addAttribute("users", Arrays.asList("user1", "user2"));
```

```html
<h5 th:each="user:${users}" th:text="${user}"></h5>
```



>  基础 语法 

- Simple expressions:
    - Variable Expressions: `${...}`
    - Selection Variable Expressions: `*{...}`
    - Message Expressions: `#{...}`
    - Link URL Expressions: `@{...}`
    - Fragment Expressions: `~{...}`
- Literals
    - Text literals: `'one text'`, `'Another one!'`,…
    - Number literals: `0`, `34`, `3.0`, `12.3`,…
    - Boolean literals: `true`, `false`
    - Null literal: `null`
    - Literal tokens: `one`, `sometext`, `main`,…
- Text operations:
    - String concatenation: `+`
    - Literal substitutions: `|The name is ${name}|`
- Arithmetic operations:
    - Binary operators: `+`, `-`, `*`, `/`, `%`
    - Minus sign (unary operator): `-`
- Boolean operations:
    - Binary operators: `and`, `or`
    - Boolean negation (unary operator): `!`, `not`
- Comparisons and equality:
    - Comparators: `>`, `<`, `>=`, `<=` (`gt`, `lt`, `ge`, `le`)
    - Equality operators: `==`, `!=` (`eq`, `ne`)
- Conditional operators: **（三元运算符，前端专用）**
    - If-then: `(if) ? (then)`
    - If-then-else: `(if) ? (then) : (else)`
    - Default: `(value) ?: (defaultvalue)`
- Special tokens:
    - No-Operation: `_`

All these features can be combined and nested:

```html
'User is of type ' + (${user.isAdmin()} ? 'Administrator' : (${user.type} ?: 'Unknown'))
```







# 8 MVC 配置原理

> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.6.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

默认原有MVC自动配置：

![image-20200630122114683](SpringBoot.assets/image-20200630122114683.png)

如果想定义一个自己的视图解析器：

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Bean
    public ViewResolver myViewResolver() {
        return new MyViewResolver();
    }

    public static class MyViewResolver implements ViewResolver {
        @Override
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            return null;
        }
    }
}
```

按照上面的描述：编写一个类，继承 WebMvcConfigurer，增加<font color='orange'>@Configuration</font>注解。



**But without @EnableWebMvc：**

添加了这个注解之后，默认的  WebMVCAutoConfiguration就全部失效了。

> 原因：
>
> @EnableWebMvc 导入了DelegatingWebMvcConfiguration
>
> DelegatingWebMvcConfiguration 继承了 WebMvcConfigurationSupport

![image-20200630125256638](SpringBoot.assets/image-20200630125256638.png)

如果存在WebMvcConfigurationSupport，那么整个WebMvcAutoConfiguration就会失效。



## 我的自定义配置类

WebMvcConfigurer接口下的所有方法:

![image-20200712191506561](SpringBoot.assets/image-20200712191506561.png)



添加自定义配置的方式：

1. 重写对应的函数
2. 编写自定义的类，在自定义配置类中注入



### configurePathMatch

这个用到的比较少，这个是和访问路径有关的。举个例子，比如说PathMatchConfigurer 有个配置是setUseTrailingSlashMatch(),如果设置为true的话（默认为true），后面加个斜杠并不影响路径访问，例如“/user”等同于“/user/"。我们在开发中很少在访问路径上搞事情，所以这个方法如果有需要的请自行研究吧。



### configureContentNegotiation

这个东西直译叫做内容协商机制，主要是方便一个请求路径返回多个数据格式。ContentNegotiationConfigurer这个配置里面你会看到MediaType，里面有众多的格式。此方法不在多赘述。



### configureAsyncSupport

顾名思义，这是处理异步请求的。只能设置两个值，一个超时时间（毫秒，Tomcat下默认是10000毫秒，即10秒），还有一个是AsyncTaskExecutor，异步任务执行器。

### configureDefaultServletHandling

这个接口可以实现静态文件可以像Servlet一样被访问。

### addFormatters

> 增加转化器或者格式化器。这边不仅可以把时间转化成你需要时区或者样式。还可以自定义转化器和你数据库做交互，比如传进来userId，经过转化可以拿到user对象。



### addInterceptors 添加拦截器

> 添加一个拦截器

```java
private LoginHandlerInterceptor loginHandlerInterceptor;

// 注入
@Autowired
public void setLoginHandlerInterceptor(LoginHandlerInterceptor loginHandlerInterceptor) {
    this.loginHandlerInterceptor = loginHandlerInterceptor;
}

/**
     * 添加拦截器
     * @param registry
     */
@Override
public void addInterceptors(InterceptorRegistry registry) {
    // 添加拦截器
    registry.addInterceptor(loginHandlerInterceptor).
        addPathPatterns("/**"). // 添加拦截路径
        excludePathPatterns("/index.html", "/", "/user/login", "/css/**", "/js/**", "/img/**");

}
```

`addPathPatterns("/**")`对所有请求都拦截，但是排除了相关静态资源文件和首页、登录页。	

自定义拦截器：

```java
@Component
public class LoginHandlerInterceptor implements HandlerInterceptor {


    /**
     * 进行逻辑判断，如果ok就返回true，不行就返回false，返回false就不会处理请求
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {


        // 在 /user/login 控制器下，如果登录成功才设置loginUser
        Object loginUser = request.getSession().getAttribute("loginUser");

        if (loginUser==null) {
            request.setAttribute("msg", "请先登录！");
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        } else {
            return true;
        }
    }
}
```

**逻辑分析：**我的拦截器拦截了除了静态资源和首页、登录页以外的所有视图！ 判断是否需要拦截的条件是有没有 loginUser这个session。



### addResourceHandlers 自定义资源映射

> 我们想自定义静态资源映射目录的话，只需重写addResourceHandlers方法即可。
>
> 注：如果继承WebMvcConfigurationSupport类实现配置时必须要重写该方法，一般不需要改。

```java
/**
 * 配置静态访问资源
 * @param registry
 */
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/my/**").addResourceLocations("classpath:/my/");
   
}
// 我们访问自定义my文件夹中的elephant.jpg 图片的地址为 http://localhost:8080/my/elephant.jpg

```
> 如果你想指定外部的目录也很简单，直接addResourceLocations指定即可

```java
@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/my/**").addResourceLocations("file:E:/my/");
        
    }
// addResourceLocations指的是文件放置的目录，addResoureHandler指的是对外暴露的访问路径
// windows当服务器的情况下，前面一定要加上一个file:。
```





### addCorsMappings 跨域映射 

> Cors = Cross-Origin resource sharing：跨域资源共享

包含了三个重要的概念：**域、资源、同源策略**。

- 域：由protocol、ip、port组成的一个站点。
- 资源：URL对应的一个内容。图片、文字、代码、JSON
- 同源策略：为了防止XSS，浏览器，客户端应该仅请求与当前页面来自同一个域的资源，请求其他域资源时需要验证。

为了让跨域请求可以正常发送，又不破坏同源策略的安全性情况下，我们需要==CORS==机制。

在客户端、浏览器发送实际请求之前，首先会发送一个预检请求：==判断能不能发送请求==。



#### 1. 注解方式 单个controller配置

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @RequestMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

```java
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @RequestMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

```kotlin
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    + @CrossOrigin(origins = "http://domain2.com")
    @RequestMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```





#### 2.全局配置

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://domain2.com")
        .allowedMethods("GET", "POST", "DELETE")
        .allowedHeaders("header1").allowCredentials(false).maxAge(3600);
    }
}
```



@Bean注入方式

```java
@Configuration
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```





注意：在引入Spring Security之后，常规的配置会错问题。==具体的原因复习印象笔记==。

> **发现ajax post跨域请求时，默认是不携带浏览器的cookie的**，也就是每次请求都会生成一个新的session，因此post请求都被登录拦截。

可以在Spring Boot应用程序中声明如下所示的过滤器：

```java
@Configuration
public class MyConfiguration {

    @Bean
    public FilterRegistrationBean corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://domain1.com");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        source.registerCorsConfiguration("/**", config);
        FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
        bean.setOrder(0);
        return bean;
    }
}
```







### addViewControllers 页面跳转

> 以前写SpringMVC的时候，如果需要访问一个页面，必须要写Controller类，然后再写一个方法跳转到页面，感觉好麻烦，其实重写WebMvcConfigurer中的addViewControllers方法即可达到效果了

```java
/**
     * 以前要访问一个页面需要先创建个Controller控制类，再写方法跳转到页面
     * 在这里配置后就不需要那么麻烦了，直接访问http://localhost:8080/toLogin就跳转到login.jsp页面了
     * @param registry
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/toLogin").setViewName("login");
        
    }
```

值的指出的是，在这里重写addViewControllers方法，并不会覆盖WebMvcAutoConfiguration中的addViewControllers（在此方法中，Spring Boot将“/”映射至index.html），这也就意味着我们自己的配置和Spring Boot的自动配置同时有效，这也是我们推荐添加自己的MVC配置的方式。

### configureViewResolvers 配置视图解析



### addArgumentResolvers



### addReturnValueHandlers



### configureMessageConverters



### extendMessageConverters



### configureHandlerExceptionResolvers



### extendHandlerExceptionResolvers



### getValidator()



### getMessageCodesResolvers







# 9 Web 开发

前端引入thymeleaf依赖：

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```



## 9.1 国际化

1. 配置 i18n文件

![image-20200630163854274](SpringBoot.assets/image-20200630163854274.png)

- 创建对应语言的properties文件，定义需要使用不同语言前端文本的字段（多语言）

- 在.html页面中配置 

    ![image-20200630165321693](SpringBoot.assets/image-20200630165321693.png)

2. 国际化文件配置

    这里涉及到SpringBoot对国际化的自动配置! 有一个类:<font color='cornflowerblue'>MessageSourceAutoConfiguration</font>.

    有属性spring.messages.basename = "messages"。设置成我们的路径：

    ![image-20200630170025167](SpringBoot.assets/image-20200630170025167.png)

3. 自定义 LocaleResolver 组件

    ```java
    public class MyLocaleResolver implements LocaleResolver {
    
    
        @Override
        public Locale resolveLocale(HttpServletRequest request) {
            // 接受来自前端的参数, lang = "zh_CN"
            String language = request.getParameter("lang");
    
            Locale locale = Locale.getDefault();
    
            if (!StringUtils.isEmpty(language)) {
                String[] s = language.split("_");
    
                locale = new Locale(s[0], s[1]);
            }
    
            return locale;
        }
    
        @Override
        public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    
        }
    }
    ```

    - 在Spring中有一个国际化的Locale （区域信息对象）；里面有一个叫做LocaleResolver （获取区域信息对象）的解析器！

    ![image-20200630170407562](SpringBoot.assets/image-20200630170407562.png)

    看到默认里，初始化了AcceptHeaderLocaleResolver，实现了LocaleResolver。那我们也跟着这个来实现一个自己的 LocaleResovler。

4. 将自定义类注册到Spring容器：@Bean

    ```java
    @Configuration
    public class MyMvcConfig implements WebMvcConfigurer {
    
        @Bean
        public LocaleResolver localeResolver() {
            return new MyLocaleResolver();
        }
    }
    ```

    

5. 前端编写跳转方式

    ```html
    <a class="btn btn-sm" th:href="@{/index.html(lang=zh_CN)}">中文</a>
    <a class="btn btn-sm" th:href="@{/index.html(lang=en_US)}">English</a>
    ```

    跳转到index.html页面并附带参数，这个参数被MyLocaleResovler截获。

    

## 9.2 页面与Controller的交互

完成登录：

- 成功登录，跳转到dashboard
- 登录失败，提示用户名密码错误

> 1、前端表单

![image-20200630203913060](SpringBoot.assets/image-20200630203913060.png)



查看表单、按钮。添加

```html
<form class="form-signin" th:action="@{/user/login}">
```

点击登录后跳转的url： /usr/login。



> 2、后端controller

添加 Logincontroller：

```java
@Controller
public class LoginController {

    @RequestMapping("/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Model model) {
        if (!StringUtils.isEmpty(username) && "123456".equals(password)) {
            return "dashboard";
        } else {
            // 登陆失败
            model.addAttribute("msg", "用户名或者密码错误！");
            return "index";
        }
    }

}
```

路由为刚才设置的 /usr/login，执行 login() 方法。 参数为从form接收的 username 和 password。继续回前端设置



> 3、前端参数传递

```html
<input type="text" name="username" class="form-control" required="" autofocus="" th:placeholder="#{login.username}">

<input type="password" name="password" class="form-control"  required="" th:placeholder="#{login.password}">
```

必须在 <input> 设置name，传递参数！

登录成功可以跳转到dashboard视图，登录失败则需要返回index视图，同时携带msg消息。

**如何显示这个msg？**

在 index.html页面 增加 消息的显示

```html
<p style="color: #ff0000" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

定义显示msg的条件：msg需要不为空，说明是尝试过登录的。

thymeleaf的条件判断语法：

- If-then: `(if) ? (then)`
- If-then-else: `(if) ? (then) : (else)`
- Default: `(value) ?: (defaultvalue)`

布尔运算符：

- Binary operators: `and`, `or`
- Boolean negation (unary operator): `!`, `not`

#strings. 是thymeleaf的string工具类。

```html
th:if="${not #strings.isEmpty(msg)}"
```



> 问题1：dashboard-url栏中出现了 表单信息

```html
http://localhost:8080/user/login?username=2018202110061&password=123456
```

因为直接返回的  dashboard 视图，所以会把信息附带在url中。

解决：<font color='orange'>加视图</font>

```java
    if (!StringUtils.isEmpty(username) && "123456".equals(password)) {
        return "redirect:/main.html";
    } else {
        // 登陆失败
        model.addAttribute("msg", "用户名或者密码错误！");
        return "index";
    }
}
```

将成功登录重定向到 main.html

同时在 自定义 MVC Config中添加视图：

![image-20200630205543224](SpringBoot.assets/image-20200630205543224.png)

这里的main.html可以用任意的字符串代替，取main也可以，甚至可以取dashboard。只要Controller和Configuration两者相互对应就可以。



> 问题2：直接在地址栏输入main.html也可以到dashboard

在Configuration中增加了视图控制后，现在不需要正确登录也能直接到dashboard.html页面了。

解决办法：<font color='orange'>加拦截器</font>

自定义拦截器： LoginHandlerInterceptor 实现 HandlerInterceptor接口

重写方法

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

	// 进行逻辑判断，如果ok就返回true，不行就返回false，返回false就不会处理请求
    return false;
}
```

**思考：如果登录成功会怎么样？**

> > 会有session！

所以只要request.getSession()能获取到session就可以了！<font color='red'>因此我们需要在登录成功的时候添加session。</font>

![image-20200630212608482](SpringBoot.assets/image-20200630212608482.png)

```html
session = <"loginUser", username>
```

- 编写拦截器:

```java
public class LoginHandlerInterceptor implements HandlerInterceptor {


    /**
     * 进行逻辑判断，如果ok就返回true，不行就返回false，返回false就不会处理请求
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {


        Object loginUser = request.getSession().getAttribute("loginUser");

        if (loginUser==null) {
            request.setAttribute("msg", "请先登录！");
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        } else {
            return true;
        }
    }
```

通过request.getSession().getAttribute() 获得对应的session，如果session存在，则返回true；否则跳转到index页面，并带一个msg消息！



- 配置拦截器

同样在自定义Configuration中，重写 addInterceptors 方法

![image-20200630213017642](SpringBoot.assets/image-20200630213017642.png)

1. 添加拦截器
2. 添加拦截的路径
3. 排除不需要拦截的资源、路径
    - /index.html   /
    - /user/login
    - 静态资源文件

效果如下：

![image-20200630213148718](SpringBoot.assets/image-20200630213148718.png)

> 回顾Spring中，request.getRequestDispatcher

- Web应用是  请求\响应 架构， 而request和response就是在服务器端生成的相应的两个对象，request能够获取客户端传递的参数以及相关的一些信息，而response就是给客户端相关的页面信息。

- request.getRequestDispatcher.forward(request.response) 表示将客户端的请求转向（forward）到getRequestDispatcher（）方法中参数定义的页面或者链接。

- ```
    说通俗点就是，当一个客户端的请求到这个页面后，不做处理或者不处理完，将请求转给另一个页面处理，然后再响应给客户端。
    ```





## 9.3 查询 + 前端展示

> dashboard.html 主要内容

![image-20200701210935423](SpringBoot.assets/image-20200701210935423.png)

包含：

- 导航栏
- 侧边栏
- 内容显示区域



> list.html 主要内容

![image-20200701211039972](SpringBoot.assets/image-20200701211039972.png)

包含：

- 导航栏
- 侧边栏
- 内容显示区域



两个页面除了内容显示区域不同之外，导航栏和侧边栏是一致的，因此代码冗余。提取出来到commoms。

> thymeleaf 中的 th:fragment 和 th:insert / replace 用于模板

创建 common/common.html

- 导航栏

![image-20200701211857740](SpringBoot.assets/image-20200701211857740.png)

- 侧边栏

![image-20200701211904555](SpringBoot.assets/image-20200701211904555.png)



dashboard.html 和 list.html 中使用

![image-20200701211952142](SpringBoot.assets/image-20200701211952142.png)



> 问题：将list.html 按照如下目录防止后，list.html无法访问common.html

![image-20200701223749688](SpringBoot.assets/image-20200701223749688.png)









点击员工管理，高亮员工管理按钮，且返回所有员工的信息。

1. 设置高亮 & 创建  路由跳转

    ![image-20200701224120912](SpringBoot.assets/image-20200701224120912.png)

2. EmployeeController

    ```java
    @Controller
    public class EmployeeController {
    
    
        @Autowired
        private EmployeeDao employeeDao;
    
        @RequestMapping("/emps")
        public String list(Model model) {
    
            Collection<Employee> employees = employeeDao.getAll();
            model.addAttribute("emps", employees);
    
            return "emps/list";
        }
    }
    ```

3. 前端显示数据

    ![image-20200701224251369](SpringBoot.assets/image-20200701224251369.png)



## 9.4 添加

> 1. 跳转controller，到添加信息的视图



- 增加 `添加员工` 按钮：

![image-20200702160150887](SpringBoot.assets/image-20200702160150887.png)

- 添加路由跳转

![image-20200702160213395](SpringBoot.assets/image-20200702160213395.png)

> > 它是一个按钮，当有跳转功能的时候，用 a 标签。

- 添加Controller

![image-20200702164848733](SpringBoot.assets/image-20200702164848733.png)

由于部门是 <select> ， 所以需要先把部门信息提取出来。 （性别就两项，不需要提取了）

add.html中，有一个添加信息的表单： action![image-20200702165023950](SpringBoot.assets/image-20200702165023950.png)

其他前端内容不赘述，部门如何展示：

![image-20200702165106891](SpringBoot.assets/image-20200702165106891.png)



![image-20200702165121319](SpringBoot.assets/image-20200702165121319.png)

通过th:each 输出从controller取到的部门信息。

> 2. 提交表单内容， 写入数据库

![image-20200702165337711](SpringBoot.assets/image-20200702165337711.png)

表单提交的method=“post”，链接到此Controller，接受前端的employee各字段数据<font color='orange'>（保证前端每个input 的 name属性和 Employee类一致）</font>

添加成功后，返回list.html



## 9.5 更新

与添加类似，步骤总结：

1. 更新按钮，跳转路由

    ```html
    <a class="btn btn-sm btn-primary" th:href="@{/emp/} + ${emp.getId()}">
       编辑
    </a>
    ```

2. Controller接受前端id字段，跳转更新页面

    ```java
    @GetMapping("/emp/{id}")
    public String toUpdatePage(Model model, @PathVariable Integer id) {
    
        // 查询原来的员工数据
        Employee emp = employeeDao.getEmployeeById(id);
    
        model.addAttribute("emp", emp);
    
    
        // 查询部门信息，需要在前端进行展示
        Collection<Department> departments = departmentDao.getDepartments();
    
        model.addAttribute("depts", departments);
    
        return "emps/update";
    }
    ```

3. 更新页面表单展示带有的信息， action=更新路由 method="post"

    1. radio![image-20200702171556278](SpringBoot.assets/image-20200702171556278.png)
    2. select![image-20200702171624130](SpringBoot.assets/image-20200702171624130.png)
    3. action![image-20200702171645874](SpringBoot.assets/image-20200702171645874.png)

4. 修改数据

    ```java
    @PostMapping("/updateEmp/{id}")
    public String updateEmp(Employee employee,  @PathVariable Integer id) {
        System.out.println("需要修改的employ 的 ID =" + id);
        employee.setId(id);
        employeeDao.save(employee);
    
        return "redirect:/emps";
    }
    ```

5. 返回list.html





## 9.6 修改



- 添加路由跳转

```html
<a class="btn btn-sm btn-danger" th:href="@{/delEmp/} + ${emp.getId()}">
   删除
</a>
```

- Controller删除数据

```java
@GetMapping("/delEmp/{id}")
public String deleteEmp(Model model, @PathVariable("id") Integer id) {

    employeeDao.deleteEmployeeById(id);
    return "redirect:/emps";
}
```



## 9.7 404 error

在template下创建error文件夹，放入404.html



## 9.8 注销



```java
@RequestMapping("/user/logout")
public String logout(HttpSession session) {
	// 干掉session
    session.invalidate();
    return "redirect:/index.html";
}
```





# 10 CommandLineRunner

> 在我们实际工作中，总会遇到这样需求，在项目启动的时候需要做一些初始化的操作，比如初始化线程池，提前加载好加密证书等。

 `CommandLineRunner`，`CommandLineRunner` 接口的 `Component` 会在所有 `Spring Beans `都初始化之后，`SpringApplication.run() `之前执行，非常适合在应用程序启动之初进行一些数据初始化的工作。



只需要实现CommandLineRunner 或者ApplicationRunner 接口。重写 run方法。 两个接口的区别是：

- CommandLineRunner. run(String... args)
- ApplicationRunner.run(ApplicationArguments args)



实现过程分三部分：

1. 实现CommandLineRunner接口
2. 添加Component注解
3. 添加Order注解



我们可以创建**多个**实现`CommandLineRunner`和`ApplicationRunner`接口的类。为了使他们按一定顺序执行，可以使用`@Order`注解或实现`Ordered`接口。

![image-20200705201734711](SpringBoot.assets/image-20200705201734711.png)

![image-20200705201748784](SpringBoot.assets/image-20200705201748784.png)







# 11 WebFlux 响应式编程



## 响应式编程 Reactive Programming

> 面向数据流和变化传播的编程范式



举例说明：

例如，在命令式编程环境中，a=b+c 表示将表达式的结果赋给 a，而之后改变 b 或 c 的值不会影响 a 。但在响应式编程中，a 的值会随着 b 或 c 的更新而更新。



**特点：**异步 +  事件驱动

我们以前编写的大部分都是阻塞类的程序，当一个请求过来时任务会被阻塞，直到这个任务完成后再返回给前端；响应式编程接到请求后只是提交了一个请求给后端，后端会再安排另外的线程去执行任务，当任务执行完成后再异步通知到前端。





# 12 ElasticSearch 整合



## 是什么

我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前全文搜索引擎的 首选。

Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用 多shard(分片)的方式保证数据安全，并且提供自动resharding的功能，github 等大型的站点也是采用了ElasticSearch作为其搜索服务。



## 基本概念

- **文档**：一个员工数据。
- **索引**：存储数据到ElasticSearch的过程。

![img](SpringBoot.assets/16932f26bb9d022c)



## 常用注解 @Document @Field

```java
public @interface Document {
 
String indexName(); //索引库的名称，个人建议以项目的名称命名
 
String type() default ""; //类型，个人建议以实体的名称命名
 
short shards() default 5; //默认分区数
 
short replicas() default 1; //每个分区默认的备份数
 
String refreshInterval() default "1s"; //刷新间隔
 
String indexStoreType() default "fs"; //索引文件存储类型
}
```



```java
public @interface Field {
 
FieldType type() default FieldType.Auto; //自动检测属性的类型，可以根据实际情况自己设置
 
FieldIndex index() default FieldIndex.analyzed; //默认情况下分词，一般默认分词就好，除非这个字段你确定查询时不会用到
 
DateFormat format() default DateFormat.none; //时间类型的格式化
 
String pattern() default ""; 
 
boolean store() default false; //默认情况下不存储原文
 
String searchAnalyzer() default ""; //指定字段搜索时使用的分词器
 
String indexAnalyzer() default ""; //指定字段建立索引时指定的分词器
 
String[] ignoreFields() default {}; //如果某个字段需要被忽略
 
boolean includeInParent() default false;
}
```



## 问题

> 保证ES的版本与SpringBoot项目中的版本一致，否则无法连接。

如何修改版本依赖：

![image-20200707111832180](SpringBoot.assets/image-20200707111832180.png)

在pom.xml中增加对应依赖的版本，就会覆盖默认的版本。



根据官方文档

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html

我们首先需要创建一个对象，注入Bean。

![image-20200707113357539](SpringBoot.assets/image-20200707113357539.png)



目前的版本7.x.x 已经不怎么使用 Jest、原生的API，基本上都使用Rest API。源码中也为我们提供了几个对象：![image-20200707121607119](SpringBoot.assets/image-20200707121607119.png)



- 查看第一个 BuilderConfiguration：

![image-20200707121910747](SpringBoot.assets/image-20200707121910747.png)

> 给了两个Bean，一些默认的配置都在这个类中构建。

- RestHighLevelClientConfiguration

![image-20200707122031901](SpringBoot.assets/image-20200707122031901.png)

> 我们常用的这个类构建自己的：例如官方文档中给出的第一个例子

```java
@Configuration
public class ElasticSearchConfig {

    /**
     * <bean id="restHighLevelClient" class="RestHighLevelClient"></>
     * @return
     */
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("192.168.1.114", 9200, "http")
                )
        );
        return client;
    }

}
```

配置了RestHighLevelClient，上面的@ConditionalOnMissingBean注解就生效了。要是想用他默认的，bean的名称就是elasticsearchRestHighLevelClient， 而我们的对象叫 ：restHighLevelClient。

- 第三个：普通的RestClient

![image-20200707122549874](SpringBoot.assets/image-20200707122549874.png)

> 平时也不用这个 Low Level的API。



## ES-Java API 测试

**一、创建索引测试**

```java
@SpringBootTest
class KiccEsApiApplicationTests {

   @Autowired
   @Qualifier("restHighLevelClient")
   private RestHighLevelClient client;

   @Test
   void testCreateIndex() throws IOException {
      // 1、创建索引请求
      CreateIndexRequest request = new CreateIndexRequest("kicc_index");
      // 2、客户端执行请求IndicesClient，请求后获得响应
      CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);

      System.out.println(createIndexResponse);
   }

}
```

打开 ES-head，连接ES，可以看到

```bash
docker run -p 9100:9100 mobz/elasticsearch-head:5
```

![image-20200707140525374](SpringBoot.assets/image-20200707140525374.png)





>  常用API

```java
package com.kicc;

import com.alibaba.fastjson.JSON;
import com.kicc.pojo.User;
import com.kicc.utils.ESCont;
import org.apache.lucene.util.QueryBuilder;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.MatchAllQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.TermQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.FetchSourceContext;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.ArrayList;
import java.util.concurrent.TimeUnit;

@SpringBootTest
class KiccEsApiApplicationTests {

   @Autowired
   @Qualifier("restHighLevelClient")
   private RestHighLevelClient client;

   @Test
   void testCreateIndex() throws IOException {
      // 1、创建索引请求
      CreateIndexRequest request = new CreateIndexRequest("kicc_index");
      // 2、客户端执行请求IndicesClient，请求后获得响应
      CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);

      System.out.println(createIndexResponse);
   }

   /**
    * 判断索引是否存在
    * @throws IOException
    */
   @Test
   void testIndexExist() throws IOException {
      GetIndexRequest request = new GetIndexRequest("kicc_index");
      boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
      System.out.println(exists?"索引存在":"索引不存在");
   }

   /**
    * 测试插入文档
    * 1、创建Index请求
    * 2、设置请求的Id，超时
    * 3、.source方法放入Json
    * 4、用client发送请求
    */
   @Test
   void testAddDocument() throws IOException {
      User user = new User("Kicc", 23);

      // 创建请求，ES中都是用Request
      IndexRequest request = new IndexRequest("kicc_index");

      // 规则  put /kicc_index/_doc/1
      request.id("1");
      request.timeout(TimeValue.timeValueSeconds(1));

      // 将我们的数据放入请求 Json; 导入 fastjson
      IndexRequest source = request.source(JSON.toJSONString(user), XContentType.JSON);

      // 客户端发送请求，获取响应结果
      IndexResponse index = client.index(request, RequestOptions.DEFAULT);

      System.out.println(index.getIndex());
      System.out.println(index.status());
      System.out.println(index.toString());
      System.out.println(index.getResult());
   }

   /**
    * 测试存在与否
    * @throws IOException
    */
   @Test
   void testIsExist() throws IOException {
      // 返回Get的Request
      GetRequest getRequest = new GetRequest("kicc_index", "1");

      // 不获取返回的 _source 的上下文 （不设置也可以）
      getRequest.fetchSourceContext(new FetchSourceContext(false));

      boolean exists = client.exists(getRequest, RequestOptions.DEFAULT);
      System.out.println(exists);
   }

   /**
    * 测试文档获取：根据id
    * 1、获取Get请求
    * 2、client发送get请求
    * 3、调用结果的各种返回：getSourceAsString
    */
   @Test
   void testGetDocument() throws IOException {
      GetRequest getRequest = new GetRequest("kicc_index", "2");
      GetResponse documentFields = client.get(getRequest, RequestOptions.DEFAULT);
      String sourceAsString = documentFields.getSourceAsString();
      System.out.println(sourceAsString);
      System.out.println(documentFields);
   }

   /**
    * 测试更新文档
    * 1、获取Update请求
    * 2、设置超时
    * 3、.doc方法添加Json字符串 （实际调用了.source()方法)
    * 4、client发送更新请求
    */
   @Test
   void testUpdateDocument() throws IOException {
      UpdateRequest updateRequest = new UpdateRequest("kicc_index", "1");
      User user = new User("凯哥", 18);

      // 超过一秒就不执行
      updateRequest.timeout("1s");
      updateRequest.doc(JSON.toJSONString(user), XContentType.JSON);

      UpdateResponse update = client.update(updateRequest, RequestOptions.DEFAULT);
      System.out.println(update);

      testGetDocument();

   }

   /**
    * 测试删除文档
    * 1、delete请求
    * 2、client发送删除请求
    */
   @Test
   void testDeleteDocument() throws IOException {
      DeleteRequest deleteRequest = new DeleteRequest("kicc_index", "1");
      deleteRequest.timeout("1s");

      DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
      System.out.println(deleteResponse);

   }

   /**
    * 大批量的插入数据
    * 1、创建bulk请求
    * 2、设置超时（根据实际请求，可以设置大一点）
    * 3、循环 add加入：每一个都是Index请求；请求中添加Json字符串
    * 4、client发送批量请求
    */
   @Test
   void testBulkRequest() throws IOException {
      BulkRequest bulkRequest = new BulkRequest("kicc_index");
      bulkRequest.timeout("10s");

      ArrayList<User> users = new ArrayList<>();
      users.add(new User("Kicc", 12));
      users.add(new User("Kicc", 13));
      users.add(new User("Kicc", 14));
      users.add(new User("Jaya", 15));
      users.add(new User("Jaya", 16));
      users.add(new User("Jaya", 18));

      for (int i = 0; i < users.size() ; i++) {
         bulkRequest.add(
               new IndexRequest(ESCont.ES_INDEX)
                     .id(""+(i+1))
                     .source(JSON.toJSONString(users.get(i)), XContentType.JSON)
         );
      }

      // 批量删除
//    for (int i = 0; i < 6; i++) {
//       bulkRequest.add(new DeleteRequest(ESCont.ES_INDEX, ""+(i+1)));
//    }

      BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);
      System.out.println(bulkResponse.hasFailures());
   }


   /**
    * 查询
    * SearchRequest 搜索请求
    * SearchSourceBuilder 条件构造
    * HighlightBuilder 构建高亮
    * TermQueryBuilder 精确查询
    * MatchAllQueryBuilder 查询全部
    * xxx QueryBuilder 对应各类查询
    * @throws IOException
    */
   @Test
   void testSearch() throws IOException {
      SearchRequest searchRequest = new SearchRequest(ESCont.ES_INDEX);

      // 构建搜索条件
      SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
      
      // 查询条件 我们使用QueryBuilders 工具 来实现
      // termQuery：精确查询
      // matchAllQuery：查询全部
//    TermQueryBuilder queryCondition = QueryBuilders.termQuery("age", "15");
      MatchAllQueryBuilder queryCondition = QueryBuilders.matchAllQuery();

      // Builder 根据查询条件  查询
      sourceBuilder.query(queryCondition);

      // 超时设置
      sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

      // 添加 查询条件 到 请求
      searchRequest.source(sourceBuilder);

      // 发送请求
      SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

      // 获得结果
      String s = JSON.toJSONString(searchResponse.getHits());
      System.out.println(s);
      System.out.println("=============================");

      for (SearchHit hit : searchResponse.getHits().getHits()) {
         System.out.println(hit.getSourceAsMap());
      }

   }

}
```





# 13 整合Mybatis

Mybatis的整合可以用原生的xml文件进行配置和CRUD操作，也可以用纯注解的方式进行开发。



## 注解方式



### 一、导入 依赖

```xml
<dependency>
   <groupId>org.mybatis.spring.boot</groupId>
   <artifactId>mybatis-spring-boot-starter</artifactId>
   <version>2.0.0</version>
</dependency>
```



### 二、连接数据库

在application.yaml\properties中定义数据库相关信息

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=admin
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```



### 三、定义POJO

```java
public class User implements Serializable {

   private static final long serialVersionUID = 1L;
   private Long id;
   private String userName;
   private String passWord;
   private UserSexEnum userSex;
   private String nickName;
```



### 四、Mapper

```java
@Mapper
@Repository
public interface UserMapper {
   
   @Select("SELECT * FROM users")
   @Results({
      // property 代表 pojo中的字段，column代表表中的字段，解决两者不统一的情况
      @Result(property = "userSex",  column = "user_sex", javaType = UserSexEnum.class),
      @Result(property = "nickName", column = "nick_name")
   })
   List<User> getAll();
   
   @Select("SELECT * FROM users WHERE id = #{id}")
   @Results({
      @Result(property = "userSex",  column = "user_sex", javaType = UserSexEnum.class),
      @Result(property = "nickName", column = "nick_name")
   })
   User getOne(Long id);

   @Insert("INSERT INTO users(userName,passWord,user_sex) VALUES(#{userName}, #{passWord}, #{userSex})")
   void insert(User user);

   @Update("UPDATE users SET userName=#{userName},nick_name=#{nickName} WHERE id =#{id}")
   void update(User user);

   @Delete("DELETE FROM users WHERE id =#{id}")
   void delete(Long id);

}
```

- ==接口==上使用@Mapper 和 @Repository 注解
- 方法上使用CRUD各自的注解
- 查询方法有返回结果：用@Results注解 property 对应 类中的属性名，column对应数据表中的列名。



### 多数据源 DataSource

- 配置 application.properties

```properties
mybatis.type-aliases-package=com.kicc.model # 用户 CRUD 别名

spring.datasource.test1.jdbc-url=jdbc:mysql://localhost:3306/test1?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.test1.username=root
spring.datasource.test1.password=admin
spring.datasource.test1.driver-class-name=com.mysql.cj.jdbc.Driver

spring.datasource.test2.jdbc-url=jdbc:mysql://localhost:3306/test2?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.test2.username=root
spring.datasource.test2.password=admin
spring.datasource.test2.driver-class-name=com.mysql.cj.jdbc.Driver
```



- 配置数据源的Config

```java
@Configuration
@MapperScan(basePackages = "com.neo.mapper.test1", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
public class DataSource1Config {

    @Bean(name = "test1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.test1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "test1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "test1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("test1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "test1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

```java
// 扫描对应的包
@MapperScan(basePackages = "com.neo.mapper.test1", sqlSessionTemplateRef  = "test1SqlSessionTemplate")
// 对应配置文件
@ConfigurationProperties(prefix = "spring.datasource.test1") 
```

- 各自配置Mapper，与单个数据源的基本不变





## xml 方式

1. 配置pojo别名路径；mapper和config的文件路径

```yaml
mybatis:
  type-aliases-package: com.kicc.pojo
  mapper-locations: classpath:mybatis/mapper/*.xml
  config-location: classpath:mybatis/mybatis-config.xml
```







# 14 文件上传





code gist

```java
private static String UPLOADED_FOLDER = "E://temp//";
try {
    // Get the file and save it somewhere
    byte[] bytes = file.getBytes();
    Path path = Paths.get(UPLOADED_FOLDER + file.getOriginalFilename());
    // return the original path
    Files.write(path, bytes);

    redirectAttributes.addFlashAttribute("message",
            "You successfully uploaded '" + file.getOriginalFilename() + "'");

} catch (IOException e) {
    e.printStackTrace();
}
```





## xml 配置方式

- 导入依赖

- 配置文件

    ```properties
    # 配置文件的位置
    mybatis.config-location=classpath:mybatis/mybatis-config.xml
    # mapper对应的位置
    mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
    # POJO
    mybatis.type-aliases-package=com.kicc.model
    
    spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    spring.datasource.username=root
    spring.datasource.password=root
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    ```

- POJO

- Mapper接口 （只写接口）

    ```java
    public interface UserMapper {
       
       List<User> getAll();
       
       User getOne(Long id);
    
       void insert(User user);
    
       void update(User user);
    
       void delete(Long id);
    
    }
    ```

- 实现类（xml）

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.neo.mapper.UserMapper" >
        <resultMap id="BaseResultMap" type="com.neo.model.User" >
            <id column="id" property="id" jdbcType="BIGINT" />
            <result column="userName" property="userName" jdbcType="VARCHAR" />
            <result column="passWord" property="passWord" jdbcType="VARCHAR" />
            <result column="user_sex" property="userSex" javaType="com.neo.enums.UserSexEnum"/>
            <result column="nick_name" property="nickName" jdbcType="VARCHAR" />
        </resultMap>
        
        <sql id="Base_Column_List" >
            id, userName, passWord, user_sex, nick_name
        </sql>
    
        <select id="getAll" resultMap="BaseResultMap"  >
           SELECT 
           <include refid="Base_Column_List" />
          FROM users
        </select>
    
        <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
            SELECT 
           <include refid="Base_Column_List" />
          FROM users
          WHERE id = #{id}
        </select>
    
        <insert id="insert" parameterType="com.neo.model.User" >
           INSERT INTO 
                  users
                  (userName,passWord,user_sex) 
               VALUES
                  (#{userName}, #{passWord}, #{userSex})
        </insert>
        
        <update id="update" parameterType="com.neo.model.User" >
           UPDATE 
                  users 
           SET 
               <if test="userName != null">userName = #{userName},</if>
               <if test="passWord != null">passWord = #{passWord},</if>
               nick_name = #{nickName}
           WHERE 
                  id = #{id}
        </update>
        
        <delete id="delete" parameterType="java.lang.Long" >
           DELETE FROM
                   users 
           WHERE 
                   id =#{id}
        </delete>
    
    </mapper>
    ```











# 15. fastJson 整合



**添加依赖：**

```xml
<!--移除默认jackson-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--fastjson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.70</version>
</dependency>
```



**配置全局：**

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {  
	/**
     * 全局配置fastJson
     * @return
     */
    @Bean
    public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        // 配置日期的输出格式， 优先级不如注解
        fastJsonConfig.setDateFormat("yyyy-MM-dd");
        //
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
//        fastJsonConfig.setCharset(Charset.forName("UTF-8"));
        // 中文乱码
        ArrayList<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.APPLICATION_JSON);
        converter.setSupportedMediaTypes(mediaTypes);

        // convert中添加配置
        converter.setFastJsonConfig(fastJsonConfig);
        // 返回converter，也就是bean
        return converter;
    }
    
}
```





## 中文乱码

1. 添加全局配置 方式一

    ```java
    @Configuration
    public class MyMvcConfig implements WebMvcConfigurer {
    
        @Bean
        public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
            FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
            FastJsonConfig fastJsonConfig = new FastJsonConfig();
            // 配置日期的输出格式， 优先级不如注解
            fastJsonConfig.setDateFormat("yyyy-MM-dd");
            //
            fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
            // 中文乱码
            ArrayList<MediaType> mediaTypes = new ArrayList<>();
            mediaTypes.add(MediaType.APPLICATION_JSON);
            converter.setSupportedMediaTypes(mediaTypes);
    
            // convert中添加配置
            converter.setFastJsonConfig(fastJsonConfig);
            // 返回converter，也就是bean
            return converter;
        }
    }
    ```

    

    

    **方式二：**	创建一个单独的配置文件，再AutoWired到全局的配置文件中。

    MyFastJsonConfig.java

    ```java
    package com.kicc.config;
    
    import java.util.ArrayList;
    
    /**
     * @author Kicc
     * @date 20/7/14 下午 2:57
     */
    @Configuration
    public class MyFastJsonConfig {
    
    
        @Bean
        public FastJsonHttpMessageConverter fastJsonHttpMessageConverter() {
            FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
            FastJsonConfig fastJsonConfig = new FastJsonConfig();
    
            // 配置日期的输出格式， 优先级不如注解
            fastJsonConfig.setDateFormat("yyyy-MM-dd");
            //
            fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
            // 中文乱码
            ArrayList<MediaType> mediaTypes = new ArrayList<>();
            mediaTypes.add(MediaType.APPLICATION_JSON);
            converter.setSupportedMediaTypes(mediaTypes);
    
            // convert中添加配置
            converter.setFastJsonConfig(fastJsonConfig);
            // 返回converter，也就是bean
            return converter;
        }
    }
    ```

    

    MyMvcConfig.java

    ```java
    public class MyMvcConfig implements WebMvcConfigurer {
    
    
        @Autowired
        private FastJsonHttpMessageConverter fastJsonHttpMessageConverter;
    
        /**
         * fastJson配置
         * @param converters
         */
        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            converters.add(fastJsonHttpMessageConverter);
        }
    }
    ```

    





1. 在每个mapping上添加字符集

    ```java
    @ResponseBody
    @RequestMapping(value = "/testUsers", method = RequestMethod.GET, produces = "application/json; charset=utf-8")
    
    @ResponseBody
    @RequestMapping(value = "/search/{keyword}/{pageNo}/{pageSize}", method = RequestMethod.GET, produces = "application/json; charset=utf-8")
    // 返回值需要是Object，对应代码中toJSON
    public Object search(@PathVariable("keyword") String keyword,
                         @PathVariable("pageNo") int pageNo,
                         @PathVariable("pageSize") int pageSize) throws Exception {
        if (pageNo<0) {
            throw new Exception("分页起始位置需要大于0");
        }
    
        if (pageSize<=0) {
            throw new Exception("分页大小建议大于等于1");
        }
    
        List<Map<String, Object>> search = contentService.search(keyword, pageNo, pageSize);
        Object s = JSON.toJSON(search);
    
        return s;
    }
    ```



## 生日字符串转Date

```java
String time = "1996-03-09";
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
Date parse = sdf.parse(time);
employee.setBirth(parse);
```





# 16 任务



一、异步任务

> 在方法上加上注解 @Async
>
> 开启@EnableAsync



二、邮件任务

导入依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-mail</artifactId>
</dependency>

```





三、定时任务

> 在方法上加上注解 @Scheduled(crod = "*/6 * * * * ?") \@Scheduled(fixedRate = 6000) 等
>
> 开启 @EnableScheduled

**cron表达式：**

秒 	分	时	日	月	周几

周几（0-7）









# X. 部署

>  依赖管理：Maven



> 开启防火墙端口 8080



## X.1 前端Vue打包

**一、安装依赖**

```bash
npm install --unsafe-perm --registry=https://registrt.npm.taobao.org
```



**二、编译打包**

```bash
npm run build:prod
```



**三、成品**

![image-20200826143832599](SpringBoot.assets/image-20200826143832599.png)



**四、nginx部署**

进入nginx目录

```bash
cd /etc/nginx
vim nginx.
```

![image-20200826144327171](SpringBoot.assets/image-20200826144327171.png)

指向我们打包生成的dist目录。



**启动nginx**

```bash
systemctl start nginx
```







## X.2 jar 包部署 （默认）

> 清理缓存 + 打包

```bash
mvn clean
mvn package
```

得到xxxx.jar



> 开启服务

```bash
nohup java -jar xxx.jar &
```



**遇到的问题**：无法连接到redis

![image-20200826150945533](SpringBoot.assets/image-20200826150945533.png)

可能的错误原因：

- **查看有没有启动Redis服务器。**
- **redis的配置application.yml（或application.properties）中
    spring.redis.timeout连接超时时间（毫秒）中设置不能为0,
    一般修改如下：spring.redis.timeout=5000。**
- **找到redis的配置文件 redis.conf ： 执行 vim redis.conf
    3.1 protected-mode yes 改为 protected-mode no (即该配置项表示是否开启保护模式，默认是开启，开启后Redis只会本地进行访问，拒绝外部访问)。
    3.2 注释掉 bin127.0.0.1 即 #bin 127.0.0.1 (ps: 不注释掉，表示指定 redis 只接收来自于该 IP 地址的请求，注释掉后，则表示将处理所有请求)。**
- **如果在Redis中没有配置requirepass ,那么在application.properties(或application.yaml)中就不要写spring.redis.password。**

我出错的原因是第四个：Spirngboot中的 `application.yml`中设置了 spring.redis.password，但是服务器上的redis没有开启密码。这样就无法连接。



## X.3 war包部署



> 修改pom.xml

![image-20200701155516250](SpringBoot.assets/image-20200701155516250.png)



![image-20200701155543223](SpringBoot.assets/image-20200701155543223.png)



> 新增SpringBootServletInitializer实现类

```java
public class SpringBootStartApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources( RuoYiApplication.class) ;

    }

}
```

> 清理缓存 + 打包

```bash
mvn clean
mvn package
```

> 就war包放入/usr/local/tomcat/WEBAPPS



> 开启服务器本地tomcat服务

开启服务后，tomcat会自动在webapps目录下，生成对应的war包文件夹。

> 问题：无法通过ip：8080 直接访问，需要 /xxx

![image-20200826154034870](SpringBoot.assets/image-20200826154034870.png)



解决：加映射 <Host> 标签内

```bash
<Context path="/" docBase="/usr/local/tomcat/tomcat9/webapps/ruoyi" reloadable="false"></Context>
```

![image-20200701173618657](SpringBoot.assets/image-20200701173618657.png)

**作用**

把webapps/ruoyi路径  映射到 /

这样再访问 ip:8080就能直接访问到ruoyi（后台）项目！

![image-20200826154026727](SpringBoot.assets/image-20200826154026727.png)



> 如果配置了前端，那么前后端的交互已经完成！
>
> 接下来，就是考虑多配置几台后端服务器来缓解压力！



## X.4 小集群

环境配置：

- 一台前端服务器：192.168.1.114
- 一台后端服务器A：192.168.1.115
- 一台后端服务器A：192.168.1.116



> 前端配置

![image-20200701173404233](SpringBoot.assets/image-20200701173404233.png)

增加命名为ruoyi的组：

- server1 ，配置流量权重为5；
- server2，配置流量权重为3；

![image-20200701173527430](SpringBoot.assets/image-20200701173527430.png)

- proxy_pass由原来的 http://192.168.1.115:8080/ 到 ruoyi



这样，通过访问前端 192.168.1.114:80，(前端去访问后台 192.168.1.115/116:8080) 就能实现所有功能。



























# 面试



### SpringBoot 自动装配的逻辑

在SpringBoot的主启动器上默认会带有一个注解  @SpringBootApplication

这个注解接口是有三个另外的注解修饰

- SpringBootConfiguration
- EnanbleAutoConfiguration
- ComponentScan

第一个SpringBootConfiguration注解 再往底层走其实就是一个@Configuration

第二个EnableAutoConfiguration是自动装配的核心。源码从外往里剖析我们 可以找到最终指向了一个方法叫做：

`loadSpringFactories`，这个方法会通过 一个 final 修饰的字符串 `FACTORIES_RESOURCE_LOCATION`来加载所有的配置。

这个 `FACTORIES_RESOURCE_LOCATION` 指向的路径是"META-INF/spring.factories" 这个是可以在SpringBoot的autoConfigure下找到的。



所有的类都是以  xxxxAutoConfiguration 这样的命名结尾的。



但并不是说在这个文件下的所有类就会都被自动装配到容器中。我们随便点开一个类就可以看到这些类上一般都会有一个 `ConditionalOnClass`的注解，说的是如果这个注解中的参数的类已经被装配进来的话，那么当前这个类也会被装配进来。相当于是一个前置的依赖。



同时，在每一个 xxxxAutoConfiguration上呢，都会存在一个 加载 对应 xxxxProperties文件的注解。对应着每一个类的属性配置。

比如说Spring的端口，是否开启缓存这些属性。



### 你如何理解 Spring Boot 配置加载顺序？

每一个SpringBoot的版本给出的配置加载顺序可能都是不同的，这个我在看早期的Springboot版本和现在2.0以后的版本时发现的。

总共有将近20种优先级不同的方式加载配置。

常见的就是：

- 从application.properties 、 application.yaml 文件中加载
- 从系统环境变量中加载
- 从命令行参数加载
- 或者是SpringTest的时候也可能添加配置



之前提到的xxxxAutoConfiguration中的所有属性配置都是可以在 `application.yaml`中进行配置的。通过的 各个properties文件，一般只要指定对应的 前缀就可以了。







