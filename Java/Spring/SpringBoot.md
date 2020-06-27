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







> ## @EnableAutoconfiguration

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







## 2.4 配置

