# SpringMVC



## 1.1 什么是MVC



MVC三层架构：

- Model
- View
- Controller



![img](SpringMVC.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy91SkRBVUtyR0M3S3dQT1BXcTAwcE1KaWFLODZsRjZCaklYVzdXbW05S1ZFVjFGWFVmSk1EMEt6dVlaN2ljNVVIZ2dzWkRBenlZeXJkNHBMdm5CSVZNNXpBLzY0MA.jpg)





## 1.2 早期架构

![img](SpringMVC.assets/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy91SkRBVUtyR0M3S3dQT1BXcTAwcE1KaWFLODZsRjZCaklXZThSUGNDVWVleG9qQmlhUHRZN0hpYlFvblMzUGRDeTk4b1YyNEYwdFlrOEl4RVVZNDNOOTNUQS82NDA.jpg)



Model1优点：架构简单，比较适合小型项目开发；

Model1缺点：JSP职责不单一，职责过重，不便于维护；





## 1.3 Servlet 是什么？

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。



Java Servlet

```java
public class HelloServlet extends HttpServlet {
    // 由于get、post只是请求实现的不同方式，可以相互调用。业务逻辑一样。

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
        System.out.println("执行doGet方法");
        PrintWriter writer = resp.getWriter();
        writer.print("Hello, Servlet");


    }
}
```

 

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.kicc.servlet.HelloServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```



配置完tomcat之后，运行后。就是一个完整的web程序。













## 1.4 带Servlet的架构

