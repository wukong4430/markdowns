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







## **2 .2 主程序**



```java
//@SpringBootApplication ： 标注这个类是一个springboot的应用
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