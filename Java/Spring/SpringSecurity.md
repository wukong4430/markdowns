# 1 SpringSecurity

> 实际开发中，安全第一！



安全框架：shiro、SpringSecurity。两者大致相同。

做两件事：认证、授权。 Authentication、Authorize。



- 功能权限
- 访问权限
- 菜单权限
- ... 拦截器，过滤器：大量原生代码 冗余









# 2 实战

## 1 环境描述



新建空的SpringBoot项目，导入一些文件。

导入thymeleaf 依赖（不导入security依赖）

```xml
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
        </dependency>

        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-java8time</artifactId>
        </dependency>

<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-starter-security</artifactId>-->
<!--        </dependency>-->
```





文件结构描述：

![image-20200714165145912](SpringSecurity.assets/image-20200714165145912.png)



导入了css、js、html文件。使用Thymeleaf引擎。关闭cache。spring.thymeleaf.cache=false;



>  测试Controller



```java
@Controller
public class RouterController {

    @RequestMapping({"/", "/index"})
    public String index() {
        return "index";
    }
}
```

访问 localhost:8080可以正常跳转到 index；==之前不小心直接导入了security的依赖，发现不能显示index.html而是必须先登录（不是login.html，security的登录)。==



index.html展示

![image-20200714165819313](SpringSecurity.assets/image-20200714165819313.png)

添加Security后预期达到的目标：

- 不允许直接跳转到index，必须先登录
- Level 1 2 3分别对应不同级别的用户，Level 1用户只能看到Level1的信息，Level 3用户可以 看到 Level 1 2 3.
- **Authentication & Authorize**



## 2 添加Security： AOP方式 （不会影响之前的任何内容）

重要的类：

- WebSecurityConfigurerAdapter：自定义Security策略  （适配器模式）
- AuthenticationManagerBuilder：自定义认证策略    （建造者模式）
- @EnableWebSecurity：开启WebSecurity模式





### 1 新增自定义配置





```java
package com.kicc.config;

import org.springframework.security.config.annotation.web.WebSecurityConfigurer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * @author Kicc
 * @date 20/7/14 下午 5:05
 */

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 链式编程
     * 授权 Authorize
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 首页所有人可以访问
        // 但是功能也只有对应有权限的人才能访问
        http.authorizeRequests()
                .antMatchers("/").hasAnyRole("vip1", "vip2", "vip3") // 第一次登录空白身份，所以直接去登录
                .antMatchers("/level1/**").hasRole("vip1")
                .antMatchers("/level2/**").hasRole("vip2")
                .antMatchers("/level3/**").hasRole("vip3");
        // 没有权限跳转到登录页面
        http.formLogin()
                // 自己的login页面
                // 登录时通过controller访问/toLogin,/toLogin返回login.html视图
                .loginPage("/toLogin")
                // 前端传过来的 name是 username
                .usernameParameter("username")
                // 密码的name是password
                .passwordParameter("password")
                // 实际用login登录，与login.html中提交的form href一致。
                .loginProcessingUrl("/login");
        
        // 开启注销, 成功跳回到首页
        http.logout().deleteCookies("remove").logoutSuccessUrl("/");
    }
}
```

![image-20200714172428536](SpringSecurity.assets/image-20200714172428536.png)

请求不允许



设置没有权限就到登录页

![image-20200714173516470](SpringSecurity.assets/image-20200714173516470.png)

查看源码发现：

```java
/**
* The most basic configuration defaults to automatically generating a login page at
* the URL "/login", redirecting to "/login?error" for authentication failure. The
* details of the login page can be found on
* {@link FormLoginConfigurer#loginPage(String)}
*/
// 没有权限的访问就到/login， 错误就去/login?error
```



**一、内存中数据认证**

Override 认证配置 Authentization

```java
/**
 * 认证，用于login登录
 * 可以从内存中直接读，正规的需要从数据中读取
 * @param auth
 * @throws Exception
 */
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
    .withUser("Kicc")
    .password("123456").roles("vip2", "vip3")
    .and()
    .withUser("root")
    .password("1112333").roles("vip1", "vip2", "vip3")
    .and()
    .withUser("guest")
    .password("123456").roles("vip1");
}
```

![image-20200714184223702](SpringSecurity.assets/image-20200714184223702.png)

出现500错误。**解决：需要添加password的加密！**



```java
/**
 * 认证
 * 可以从内存中直接读，正规的需要从数据中读取
 * @param auth
 * @throws Exception
 */
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()) // 任选一种加密方式
    .withUser("Kicc")
    .password(new BCryptPasswordEncoder().encode("123456")).roles("vip2", "vip3")
    .and()
    .withUser("root")
    .password(new BCryptPasswordEncoder().encode("123456")).roles("vip1", "vip2", "vip3")
    .and()
    .withUser("guest")
    .password(new BCryptPasswordEncoder().encode("123456")).roles("vip1");
}
```



**二、数据库中数据认证**



```java
@Autowired
private DataSource dataSource;
 
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth)
  throws Exception {
    auth.jdbcAuthentication()
      .dataSource(dataSource)
      .withDefaultSchema()
      .withUser(User.withUsername("user")
        .password(passwordEncoder().encode("pass"))
        .roles("USER"));
}
 
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```





> 还有一个问题：进入首页后，Vip1的用户应该只能去看level1的内容。现在是level1 2 3都展示出来了。

![image-20200714192335837](SpringSecurity.assets/image-20200714192335837.png)

这个是前端的显示功能，与thymeleaf相关。需要用到thymeleaf和security的整合。



### 2 整合thymeleaf



导入依赖

```xml
<!--spring security 和 thymeleaf的整合-->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```



前端修改：

- 导入命名空间

    ```html
    <html lang="en" xmlns:th="http://www.thymeleaf.org"
          xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
    ```

- 认证

    ```html
    <!--登录注销-->
    <div class="right menu">
    
        <!--如果未登录-->
        <div sec:authorize="!isAuthenticated()"> // 如果没有登录
            <!--未登录：显示登录-->
            <a class="item" th:href="@{/toLogin}">
                <i class="address card icon"></i> 登录
            </a>
        </div>
    
    
        <div sec:authorize="isAuthenticated()"> // 如果已经登录
            <a class="item">
                用户名：<span sec:authentication="name"></span>
                &nbsp; 角色（权限）：<span sec:authentication="principal.authorities"></span>
            </a>
        </div>
        
        <div sec:authorize="isAuthenticated()">
            <!--如果已经登录：显示用户名，注销-->
            <a class="item" th:href="@{/logout}">
                <i class="sign-out icon"></i> 注销
            </a>
        </div>
    ```

    

![image-20200714195023608](SpringSecurity.assets/image-20200714195023608.png)



**只显示对应等级的内容：**

```html
<!-- 验证权限：只有是vip1的用户才能显示这个div-->
<!-- 这样就实现了同一个页面的动态展示！-->
<div class="column" sec:authorize="hasRole('vip1')">
    <div class="ui raised segment">
        <div class="ui">
            <div class="content">
                <h5 class="content">Level 1</h5>
                <hr>
                <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
            </div>
        </div>
    </div>
</div>

<!-- 验证权限：只有是vip2的用户才能显示这个div-->
<!-- 这样就实现了同一个页面的动态展示！-->
<div class="column" sec:authorize="hasRole('vip1')">
    <div class="ui raised segment">
        <div class="ui">
            <div class="content">
                <h5 class="content">Level 1</h5>
                <hr>
                <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
            </div>
        </div>
    </div>
</div>

<!-- 验证权限：只有是vip3的用户才能显示这个div-->
<!-- 这样就实现了同一个页面的动态展示！-->
<div class="column" sec:authorize="hasRole('vip1')">
    <div class="ui raised segment">
        <div class="ui">
            <div class="content">
                <h5 class="content">Level 1</h5>
                <hr>
                <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
            </div>
        </div>
    </div>
</div>
```



![image-20200714195234412](SpringSecurity.assets/image-20200714195234412.png)





### 3 常用功能



- 记住我 Remember me

    ![image-20200714200607166](SpringSecurity.assets/image-20200714200607166.png)

    开启了记住我功能，那么会在本地产生一个两周时长的 cookie

    ![image-20200714200552601](SpringSecurity.assets/image-20200714200552601.png)



​		接受前端的remember me 参数

![image-20200714202834131](SpringSecurity.assets/image-20200714202834131.png)

![image-20200714202843153](SpringSecurity.assets/image-20200714202843153.png)



总结：

- 认证
    - 通过数据库读取用户信息进行认证
- 授权：
    - 登录
    - 注销
    - 根据权限显示页面



Spring Security 帮我们把登录注销方面的活都干了。我们也不需要自己写这方面的拦截器。













# 3 Shiro

> 支持的功能

1. Authentication：身份认证/登录，验证用户是不是拥有相应的身份。
2. Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限。
3. Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的。
4. Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储。
5. Web Support：Web支持，可以非常容易的集成到 web 环境。
6. Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率。
7. Concurrency：shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去。
8. Testing：提供测试支持。
9. Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问。
10. Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。



> 运行原理

![img](https://user-gold-cdn.xitu.io/2019/9/25/16d68a13db98a523?imageslim)





> QuickStart

```java
// 常用方法
Subject currentUser = SecurityUtils.getSubject();
Session session = currentUser.getSession();
currentUser.isAuthenticated();
currentUser.getPrincipal();
currentUser.hasRole("schwartz");
currentUser.isPermitted("lightsaber:wield");
currentUser.logout();
```







# Shiro-Springboot 整合

导入依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.2.5</version>
</dependency>
```



> Shiro 中 三个核心： Subject、SecurityManager、Realm



先写个自定义CustomRealm.java

```java
// 继承 AuthorizingRealm
public class CustomRealm extends AuthorizingRealm {
    private UserMapper userMapper;

    // 需要从数据库读取 role的信息！
    @Autowired
    private void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    /**
     * 获取身份验证信息
     * Shiro中，最终是通过 Realm 来获取应用程序中的用户、角色及权限信息的。
     *
     * @param authenticationToken 用户身份信息 token
     * @return 返回封装了用户信息的 AuthenticationInfo 实例
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("————身份认证方法————");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        // 从数据库获取对应用户名密码的用户
        String password = userMapper.getPassword(token.getUsername());
        if (null == password) {
            throw new AccountException("用户名不正确");
        } else if (!password.equals(new String((char[]) token.getCredentials()))) {
            throw new AccountException("密码不正确");
        }
        return new SimpleAuthenticationInfo(token.getPrincipal(), password, getName());
    }

    /**
     * 获取授权信息，给予他特定的权限：Admin，User，Guest
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("————权限认证————");
        String username = (String) SecurityUtils.getSubject().getPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //获得该用户角色
        String role = userMapper.getRole(username);
        Set<String> set = new HashSet<>();
        //需要将 role 封装到 Set 作为 info.setRoles() 的参数
        set.add(role);
        //设置该用户拥有的角色
        info.setRoles(set);
        return info;
    }
}

```

重写的两个方法分别是实现身份认证以及权限认证，shiro 中有个作登陆操作的 `Subject.login()` 方法，当我们把封装了用户名，密码的 token 作为参数传入，便会跑进这两个方法里面（不一定两个方法都会进入）

其中 doGetAuthorizationInfo 方法只有在需要权限认证时才会进去，比如前面配置类中配置了 `filterChainDefinitionMap.put("/admin/**", "roles[admin]");` 的管理员角色，这时进入 /admin 时就会进入 doGetAuthorizationInfo 方法来检查权限；而 doGetAuthenticationInfo 方法则是需要身份认证时（比如前面的 `Subject.login()` 方法）才会进入

再说下 UsernamePasswordToken 类，我们可以从该对象拿到登陆时的用户名和密码（登陆时会使用 `new UsernamePasswordToken(username, password);`），而 get 用户名或密码有以下几个方法

```
token.getUsername()  //获得用户名 String
token.getPrincipal() //获得用户名 Object 
token.getPassword()  //获得密码 char[]
token.getCredentials() //获得密码 Object
```





二、主体配置，三个Bean

```java
@Configuration
public class ShiroConfig {

    // 1. ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();

        // 设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);

        // 添加过滤器
        LinkedHashMap<String, String> map = new LinkedHashMap<>();
        map.put("/user/add", "authc");
        map.put("/user/update", "authc");

        bean.setFilterChainDefinitionMap(map);
        bean.setLoginUrl("/tologin");

        return bean;
    }


    // 2. DefaultWebSecurityManager
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

        // 关联 UserRealm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    // 3. realm 对象
    @Bean(name = "userRealm")
    public UserRealm getUserRealm() {
        return new UserRealm();
    }

}
```

具体的过滤策略都写到getShiroFilterFactoryBean方法中。







> Shiro 的内置过滤器 集合。

| Filter       | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| anon         | 无参，开放权限，可以理解为匿名用户或游客。**所有人可以访问** |
| authc        | 无参，需要认证                                               |
| logout       | 无参，注销，执行后会直接跳转到`shiroFilterFactoryBean.setLoginUrl();` 设置的 url |
| authcBasic   | 无参，表示 httpBasic 认证                                    |
| user         | 无参，表示必须存在用户，当登入操作时不做检查                 |
| ssl          | 无参，表示安全的URL请求，协议为 https                        |
| perms[user]  | 参数可写多个，表示需要某个或某些权限才能通过，多个参数时写 perms["user, admin"]，当有多个参数时必须每个参数都通过才算通过 |
| roles[admin] | 参数可写多个，表示是某个或某些角色才能通过，多个参数时写 roles["admin，user"]，当有多个参数时必须每个参数都通过才算通过 |
| rest[user]   | 根据请求的方法，相当于 perms[user:method]，其中 method 为 post，get，delete 等 |
| port[8081]   | 当请求的URL端口不是8081时，跳转到schemal://serverName:8081?queryString 其中 schmal 是协议 http 或 https 等等，serverName 是你访问的 Host，8081 是 Port 端口，queryString 是你访问的 URL 里的 ? 后面的参数 |

常用的主要就是 anon，authc，user，roles，perms 等














