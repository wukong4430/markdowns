## 1、Spring



- Spring是一个开源的免费容器（框架）
- Spring是一个轻量级的、非入侵式的框架
- 控制反转，面向切面编程
- 支持事务的处理，对框架整合的支持

总之，Spring是一个轻量级的IOC和面向切面的编程框架。主要用来解决企业应用开发的复杂性。

Spring是一个基于IOC和AOP的结构J2EE系统的框架

IOC 反转控制 是Spring的基础，Inversion Of Control





## 2、IOC Inversion of Control

简单说就是创建对象由以前的程序员自己new 构造方法来调用，变成了交由Spring创建对象

### 思想逻辑

-  在传统的做法中，Dao编写是JavaBean。程序修改的主动权是在程序员手中，因此当需求发生变化时，程序员需要手动地去修改代码。

- ```java
  public class UserServiceImpl implements UserService{
      private UserDao userDao;
  
      // 利用set实现动态值的注入
      public void setUserDao(UserDao userDao) {
          this.userDao = userDao;
      }
  
      public void getUser() {
          userDao.getUser();
      }
  }
  ```

  

- 之后用了 set注入的方式，程序不再具有主动性，而是变成l被动接受的对象。这是最初的IoC思想。

#### Spring 究竟干了啥子

>用了Spring之后，对象的创建、管理、控制都交给Spring来完成。这就是Ioc的本质。程序员也不需要去修改程序代码，用户的需求改变也不需要去main中传入什么参数去获取不同的service。只需要在xml文件中去修改就行了。

### IOC创建对象的方式
1.使用无参数构造方法创建对象，默认！
2.假设我们要使用有参数构造，在xml里需要把property改为constructor-arg

#### xml 文件注入 格式

```xml
    <bean id="user" class="com.kicc.pojo.User">
        <constructor-arg index="0" value="Kicc"/>
        <constructor-arg index="1" value="25"/>
    </bean>
```



#### 用过annotation方式注入
我觉得可读性不是很强，不如多写一点xml。

不过评论里说，每种注解方有各自得应用场景。（后续补充


### 与传统方式比较

#### 传统方式

通过new 关键字主动创建一个对象

#### IOC方式
对象的生命周期由Spring来管理，直接从Spring那里去获取一个对象。 IOC是反转控制 (Inversion Of Control)的缩写，就像控制权从本来在自己手里，交给了Spring。


#### 打个比喻：
- 传统方式：相当于你自己去菜市场new 了一只鸡，不过是生鸡，要自己拔毛，去内脏，再上花椒，酱油，烤制，经过各种工序之后，才可以食用。
- 用 IOC：相当于去馆子(Spring)点了一只鸡，交到你手上的时候，已经五味俱全，你就只管吃就行了。


## Spring 配置

 ### 别名 alias


<alias name="myApp-dataSource" alias="subsystemA-dataSource"/> 

 ### Bean的配置
```xml
<!-- 
    id: bean 的唯一标识符，也就是相当于我们学的对象名
    class：bean对象所对应的全限定名：包名 + 类型
    name：也是别名，而且name可以同时取多个别名
-->
    <bean id="user" class="com.kicc.pojo.User" name="user2,user3 user4;user5">
        <constructor-arg index="0" value="Kicc"/>
        <constructor-arg index="1" value="25"/>
    </bean>
```



 ### import 

 用于团队开发，多个人开发了多个类。也写了多个beans.xml文件。
 最终创建一个总的xml文件，然后将所有的xml导入。



ApplicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="beans.xml"/>
    <import resource="userBeans.xml"/>


</beans>
```





## 4、DI 依赖注入 Dependency Inject. 

简单地说就是拿到的对象的属性，已经被注入好相关值了，直接使用即可。

### 4.1、构造器注入
上面讲的beans.xml就是构造器注入，只要有构造方法就可以用。

### 4.2、Set方法注入【重点】

- 依赖注入：set注入
    - 依赖：bean对象的创建依赖与容器
    - 注入：bean对象中的所有属性，由容器来注入

【环境搭建】
1. 复杂类型

    ```java
    public class Address {
        private String address;
    
        public String getAddress() {
            return address;
        }
    
        @Override
        public String toString() {
            return "Address{" +
                    "address='" + address + '\'' +
                    '}';
        }
    
        public void setAddress(String address) {
            this.address = address;
        }
    }
    ```

    

2. 真实测试对象

    ```java
    public class Student {
        private String name;
        private Address address;
    
        private String[] books;
        private List<String> hobbies;
        private Map<String, String> card;
        private Set<String> games;
    
        private Properties info;
        private String wife = null;
    ```

3. beans.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <!--    <bean id="user" class="com.kicc.pojo.User">-->
        <!--&lt;!&ndash;        参数下标赋值&ndash;&gt;-->
        <!--        <constructor-arg index="0" value="Kiccccc"/>-->
        <!--    </bean>-->
        <bean id="student" class="com.kicc.pojo.Student">
            <property name="name" value="凯哥"/>
            <!--引用其他bean注入-->
            <property name="address" ref="address"/>
            <!--数组方式注入-->
            <property name="books">
                <array>
                    <value>红楼梦</value>
                    <value>西游记</value>
                    <value>三国演义</value>
                    <value>水浒传</value>
                </array>
            </property>
    
            <!--List方式注入-->
            <property name="hobbies">
                <list>
                    <value>吉他</value>
                    <value>写字</value>
                    <value>游泳</value>
                    <value>编程</value>
                </list>
            </property>
    
            <!--Map注入-->
            <property name="card">
                <map>
                    <entry key="身份证" value="330224"/>
                    <entry key="手机号" value="1592743"/>
                </map>
            </property>
    
            <!--Set注入-->
            <property name="games">
                <set>
                    <value>LOL</value>
                    <value>AOA</value>
                    <value>DOTA</value>
                </set>
            </property>
    
            <!--null注入-->
            <property name="wife">
                <null></null>
            </property>
    
            <!--Properties-->
            <property name="info">
                <props>
                    <prop key="学号">2018202110061</prop>
                    <prop key="生日">19960309</prop>
                    <prop key="username">Kicc</prop>
                    <prop key="password">123456</prop>
                </props>
            </property>
    
    
        </bean>
    
        <bean id="address" class="com.kicc.pojo.Address">
            <property name="address" value="第五大道"/>
        </bean>
    
    </beans>
    ```

    测试

    ```java
    public class MyTest {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    
            Student student = (Student) context.getBean("student");
            String name = student.toString();
            System.out.println("name = " + name);
    
            /*name = Student
            {name='凯哥',
            address=Address{address='第五大道'},
            books=[红楼梦, 西游记, 三国演义, 水浒传],
            hobbies=[吉他, 写字, 游泳, 编程],
            card={身份证=330224, 手机号=1592743},
            games=[LOL, AOA, DOTA],
            info={学号=2018202110061, 生日=19960309, password=123456, username=Kicc},
            wife='null'}
            */
        }
    ```

    

4. 

###  4.3、拓展方法注入

我们可以使用p和c命名空间进行注入。

官方解释：![image-20200608142321068](What is Spring.assets/image-20200608142321068.png)



userBeans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--    <bean id="user" class="com.kicc.pojo.User">-->
    <!--&lt;!&ndash;        参数下标赋值&ndash;&gt;-->
    <!--        <constructor-arg index="0" value="Kiccccc"/>-->
    <!--    </bean>-->

    <!--p命名空间注入，可以直接注入属性的值：properties-->
    <bean id="user" class="com.kicc.pojo.User" p:name="沈晨凯" p:age="24"/>

    <!--c命名空间注入，通过构造器注入（有参数构造器）：constructor-arg-->
    <bean id="user2" class="com.kicc.pojo.User" c:name="凯哥" c:age="24"/>


</beans>
```



测试

用set方式获得 property

```java
@Test
public void test2() {
    ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
    User user =  context.getBean("user", User.class);
    System.out.println(user);
	}
}
```



用构造器方式c命名空间注入

```java
@Test
public void test2() {
    ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
    User user =  context.getBean("user2", User.class);
    System.out.println(user);
	}
}
```



注意点：p命名空间和c命名空间不能直接使用。需要导入约束。

```xml
xmlns:p="http://www.springframework.org/schema/p"
xmlns:c="http://www.springframework.org/schema/c"
```



### 4.4、Bean的作用域

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |



1、Singleton 全局只有一个实例 （默认机制）

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```



2、Prototype 每次从容器中取对象都会是一个新的对象

````xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
````



3、其余request、session、application、这些个只能在web开发中使用到。





## 5、Bean的自动装配

- 自动装配是Spring满足bean依赖的一种方式！
- Spring会在上下文中自动寻找，并自动装配属性！



Spring中有三种自动装配方式

1. 在xml中显式地配置
2. 在java中显式地配置
3. 隐式地自动装配bean【重要】



### 7.1、测试

1. 环境搭建
    - 一个人有两个宠物



### 7.2、byName 自动装配

```xml
    <bean id="cat" class="com.kicc.pojo.Cat"/>
    <bean id="dog" class="com.kicc.pojo.Dog"/>

    <!--autowire byName: 会自动在上下文中查找，和自己对象set方法后面的值相对应的 bean id
                 byType: 会自动在上下文中查找，和自己对象属性类型相对应的 bean id。必须保证只有一个。
    -->
    <bean id="kai" class="com.kicc.pojo.Person" autowire="byName">
        <property name="name" value="凯哥"/>

    </bean>
```



### 7.3、 byType 自动装配

```xml
    <bean class="com.kicc.pojo.Cat"/>
    <bean class="com.kicc.pojo.Dog"/>

    <!--autowire byName: 会自动在上下文中查找，和自己对象set方法后面的值相对应的 bean id
                 byType: 会自动在上下文中查找，和自己对象属性类型相对应的 bean id。必须保证只有一个。
    -->
    <bean id="kai" class="com.kicc.pojo.Person" autowire="byType">
        <property name="name" value="凯哥"/>

    </bean>
```



小结：

- byName的时候，需要保证所有的bean的id唯一，并且这个bean需要和自动注入的属性的set方法的值一致！
- byType的时候，需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性的类型一致！

### 7.4、使用注解实现自动装配



使用注解必须：

1. 导入约束：context约束

2. 配置注解的支持

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            https://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            https://www.springframework.org/schema/context/spring-context.xsd">
    
        <context:annotation-config/>
    
    </beans>
    ```



@Autowired

直接在属性上使用即可！也可以在set方法上使用！

使用Autowired我们可以不用编写Set方法。（反射）前提是你这个自动装配的属性在IOC容器中存在，且符合名字byName



科普

```java
@NUllable 字段标记了这个注解，说明这个字段可以为null
```

```java
public @interface Autowired {
    boolean required() default true;
}
```



测试代码

```java
public class Person {
    private String name = "凯哥";
    // 如果required = fasle, 传入的是null或者不传也行
    @Autowired(required = false)
    private Dog dog;
    @Autowired
    private Cat cat;
```



如果@Autowired自动装配的环境比较复杂，自动装配无法通过一个注解【@Autowired】完成的时候、我们可以使用@Qualifier(value = "xxx") 去配置@Autowired的使用，指定一个唯一的bean对象注入！



```java
@Autowired
@Qualifier(value = "cat22")
private Cat cat;
```



@Resource注解

```java
public class Person {
    private String name = "凯哥";
    // 如果required = fasle, 传入的是null或者不传也行

    @Resource
    private Dog dog;
    
    @Resource(name = "cat22")
    private Cat cat;
```



小结：

@Resource和@Autowired的区别：

- 都是用来自动装配的，都可以放在属性字段上
- @Autowired 通过byTye的方式实现，而且必须要求这个对象存在。
- @Resource默认通过byName方式实现，如果找不到名字，则通过byType实现。如果两个都找不到，就报错。
- 执行的顺序不同：@Autowired通过byType的方式实现。@Resource默认通过byName的方式实现。





## 6、使用注解开发



在Srping4之后，要使用注解开发，必须要保证aop的包导入



### 6.1、bean



### 6.2、属性如何注入

```java	
@Component
public class User {
    private String name;
    
    // 相当于<property name="name" value="kicc"/>
    @Value("kicc")
    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```



### 6.3、衍生的注解

@Component有几个衍生的注解。在web开发中，会按照mcvc三层架构分层

- dao 【@Repository】

- service 【@Service】

- controller 【@Controller】

    这四个注解的功能是一致的，都是代表将某个类注册到Spring中，装配Bean

### 6.4、自动装配置

```
- @Autowired: 自动装配通过类型，名字。
    如果Autowired不能唯一自动装配属性，则需要通过@Qualifier(value = "xxx")
- @Nullable  字段标记了这个注解，说明这个字段可以为null；
- @Resource:  自动装配通过名字，类型。
- @Component: 组件，放在类上。说明这个类已经被Spring管理了。
```



### 6.5、作用域

```java
@Component
@Scope("singleton")
public class User {
    private String name;

    // 相当于<property name="name" value="kicc"/>
    @Value("kicc")
    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```



### 6.6、小结

xml 和 注解:

- xml 万能的，适用于任何场景。维护简单方便。
- 注解不是自己类用不了，维护相对复杂。

xml和注解的最佳实践：

- xml用来管理bean
- 注解只负责完成属性的注入
- 我们在使用的过程中，只需要注意一个问题：必须让注解生效，就需要开启注解的支持

```xml
<context:component-scan base-package="com.kicc.dao"/>
<context:annotation-config/>
```





## 7、 使用Java的方式配置Spring（抛弃xml）

我们现在要完全不适用Spring的xml配置，全权交给java来做

JavaConfig是Spring的一个子项目



实体类

```java
// Component注解的意思是这个类已经被Spring托管，注册到容器中了。
@Component
public class User {
    private String name;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    @Value("kaige") //注入属性值
    public void setName(String name) {
        this.name = name;
    }
}
```



配置类



```java
package com.kicc.config;

import com.kicc.pojo.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

/**
 * @author Kicc on 20/6/9.
 */
@Configuration
@ComponentScan("com.kicc.pojo")
@Import(KiccConfig2.class)
public class KiccConfig {

    /**
     * 注册一个bean，就相当于之前写的一个bean标签
     * 这个方法的名字，= bean.id
     * 方法的返回值, = bean.class
     * @return
     */
    @Bean
    public User getUser () {
        return new User();
    }
}
```



测试类

```java
public class MyTest {
    public static void main(String[] args) {
        // 如果完全使用了配置类方式去做，我们就只能通过AnnotationConfig上下文来获取容器，通过配置类的class对象加载！
        ApplicationContext context = new AnnotationConfigApplicationContext(KiccConfig.class);
        User getUser = context.getBean("getUser", User.class);
        System.out.println(getUser.getName());
    }
}
```



这种纯Java的配置方法，在SpringBoot中随处可见。





## 8、代理模式





### 8.1、静态代理

角色分析：

- 抽象角色： 一般会使用接口或者抽象类来解决
- 真实角色： 被代理的角色
- 代理角色： 代理真实角色，代理真实角色后，我们一般会做一些附属操作
- 客户：访问代理对象的人！





代码步骤：

 1. 接口

    ```java
    //租房接口
    public interface Rent {
        /**
         * 出租
         */
        public void rent();
    }
    ```

 2. 真实角色

    ```java
    // 房东
    public class Landlord implements Rent {
    
    
        public void rent() {
            System.out.println("房东要出租房子");
        }
    }
    ```

 3. 代理角色

    ```java
    public class Proxy {
        private Landlord landlord;
    
        public Proxy(){}
    
        public Proxy(Landlord landlord) {
            this.landlord = landlord;
        }
    
        public void rent() {
            seeHouse();
            landlord.rent();
            contract();
            fare();
        }
    
        public void seeHouse() {
            System.out.println("中介带你看房");
        }
    
        public void contract() {
            System.out.println("签合同");
        }
        public void fare() {
            System.out.println("收中介费");
        }
    }
    ```

    

 4. 客户端访问代理角色

    ```java
    public class Client {
        public static void main(String[] args) {
            //房东要租房子
            Landlord landlord = new Landlord();
            //代理，中介帮房东租房子。但是呢？代理角色一般会有附属操作。
            Proxy proxy = new Proxy(landlord);
            proxy.rent();
        }
    }
    ```

    



代理模式的好处：

- 可以使真实角色的操作更加纯粹 （房东租房）不需要去关注一些公共业务
- 公共就交给了代理角色。实现了业务的分工。
- 公共业务发生扩展的时候，方便集中管理

缺点：

- 一个真实角色就会产生一个代理角色；代码量就多了。开发效率变低。



### 10.2、加深理解

接口

```java
public interface UserService {
    public void add();
    public void delelte();
    public void update();
    public void query();
}
```



实现类

```java
public class UserServiceImpl implements UserService {

    public void add() {
        System.out.println("增加了一个用户");
    }

    public void delelte() {
        System.out.println("删除了一个用户");

    }

    public void update() {
        System.out.println("修改了一个用户");

    }

    public void query() {
        System.out.println("查询了一个用户");

    }
}
```



现在想添加一个log，则利用代理类

```java
public class UserServiceProxy implements UserService {

    private UserServiceImpl userService;

    public void setUserService(UserServiceImpl userService) {
        this.userService = userService;
    }

    public void add() {
        log("add");
        userService.add();
    }

    public void delelte() {
        log("delete");

        userService.delelte();
    }

    public void update() {
        log("update");

        userService.update();
    }

    public void query() {
        log("query");

        userService.query();
    }

    public void log(String msg) {
        System.out.println("[Debug] 使用了" + msg);
    }
}
```



测试类（没有用代理）这种情况下，如果想增加log，就只能去改变原来代码的逻辑（很不好

```java
public class Client {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();

        userService.add();
    }
}
```



测试类（用了代理）

```java
public class Client {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserServiceProxy proxy = new UserServiceProxy();
        proxy.setUserService((UserServiceImpl) userService);

        proxy.add();
    }
}
```



在不改变原来业务代码的基础上，可以添加一些新的功能。有点类似与Python的装饰器。

在公司开发中，为增加新的功能而改动原来可行的代码是大忌。

![image-20200609161707200](What is Spring.assets/image-20200609161707200.png)



业务的开发一般按照纵向发展，从数据层一直到前端。后续开发中想要添加新的功能或做出一点修改。那么就需要面向切面编程。（横向） AOP



### 10.3、动态代理





## AOP Aspect Oriented Program	

首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

- 所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务
- 所谓的周边功能，比如性能统计，日志，事务管理等等


周边功能在Spring的面向切面编程AOP思想里，即被定义为<u>**切面**</u>

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别**独立进行开发**
然后把切面功能和核心业务功能 "**编织**" 在一起，这就叫AOP

### 共2类业务
![c4fd3d0d3e559d5aaf27e89d89311ea0.png](en-resource://database/3470:1)