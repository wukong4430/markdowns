# 环境准备



- 实体类

    ```java
    public class User {
    
        private Integer id;
        private String username;
    
    }
    ```

- Service

    ```java
    public interface IUserService {
    
        public User findUserById(Integer id) ;
    }
    ```

    ```java
    public class UserServiceImpl implements IUserService {
    
    
        public User findUserById(Integer id) {
            if (id > 100) {
                return new User(id, "Lena");
            } else {
                return new User(id, "Anderson");
            }
    
        }
    }
    ```

    





# 1 原始的Socket连接访问



- Server

    ```java
    public class Server {
    
        private static boolean running = true;
    
        public static void main(String[] args) {
            try {
                ServerSocket ss = new ServerSocket(8888);
                System.out.println("===============> 开始监听");
                while (running) {
                    Socket s = ss.accept();
                    InetAddress inetAddress = s.getInetAddress();
                    System.out.println("接收到来自ip = " + inetAddress + "的请求！");
                    process(s);
                    s.close();
                }
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                System.out.println("已经返回！");
            }
        }
    
        public static void process(Socket socket) throws IOException {
            InputStream inputStream = socket.getInputStream();
            OutputStream outputStream = socket.getOutputStream();
            DataInputStream dataInputStream = new DataInputStream(inputStream);
            DataOutputStream dataOutputStream = new DataOutputStream(outputStream);
    
            // 接收来自客户端的请求
            int id = dataInputStream.readInt();
            UserServiceImpl userService = new UserServiceImpl();
            User user = userService.findUserById(id);
            // 将获得到的数据返回给Client
            dataOutputStream.writeInt(user.getId());
            dataOutputStream.writeUTF(user.getUsername());
            dataOutputStream.flush();
    
            dataInputStream.close();
            dataOutputStream.close();
    
        }
    }
    ```

- Client

    ```java
    public class Client {
    
    
        public static void main(String[] args) throws IOException {
            Socket s = new Socket("localhost", 8888);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            DataOutputStream dos = new DataOutputStream(baos);
            // 底层调用了baos.write
            dos.writeInt(12);
    
            // 向服务器端发送请求
            s.getOutputStream().write(baos.toByteArray());
            s.getOutputStream().flush();
    
            // 接受来自服务端的信息
            DataInputStream dis = new DataInputStream(s.getInputStream());
    
            int id = dis.readInt();
            String username = dis.readUTF();
            User user = new User(id, username);
            System.out.println("得到了一个User：");
            System.out.println(user);
    
            dos.close();
            dis.close();
            s.close();
    
    
    
        }
    }
    ```

> 这是最底层的通信方式，归根结底都是通过网络将数据转成二进制进行交互！



# 2 对客户端隐藏网络请求细节

- Client

    ```java
    public class Client {
        public static void main(String[] args) throws IOException {
            Stub stub = new Stub();
            System.out.println(stub.findUserById(123));
        }
    }
    ```

- Stub （代理、桩）

    ```java
    public class Stub {
    
    
        public User findUserById(Integer id) throws IOException {
            Socket s = new Socket("localhost", 8888);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            DataOutputStream dos = new DataOutputStream(baos);
            // 底层调用了baos.write
            dos.writeInt(id);
    
            // 向服务器端发送请求
            s.getOutputStream().write(baos.toByteArray());
            s.getOutputStream().flush();
    
            // 接受来自服务端的信息
            DataInputStream dis = new DataInputStream(s.getInputStream());
    
            int readInt = dis.readInt();
            String username = dis.readUTF();
            User user = new User(readInt, username);
    
            // dos会自动帮助关闭baos的流，手动关闭也不会出错
            dos.close();
            s.close();
            dis.close();
    
    
            return user;
    
        }
    }
    ```

> 到了这一层，至少Client已经不需要去管Socket啦！
>
> 但是，这里只能访问一个类，太弱了吧。整个动态代理！





# 3 启用动态代理



- Stub

    ```java
    public class Stub {
    
        public static IUserService getStub() {
            InvocationHandler handler = new InvocationHandler() {
    
                /**
                 * 定义处理器，方法实际执行者
                 * @param proxy 被代理的对象
                 * @param method 被代理对象的实际执行方法
                 * @param args 方法的参数
                 * @return
                 * @throws Throwable
                 */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    Socket s = new Socket("localhost", 8888);
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    DataOutputStream dos = new DataOutputStream(baos);
                    // 底层调用了baos.write
                    dos.writeInt(123);
    
                    // 向服务器端发送请求
                    s.getOutputStream().write(baos.toByteArray());
                    s.getOutputStream().flush();
    
                    // 接受来自服务端的信息
                    DataInputStream dis = new DataInputStream(s.getInputStream());
    
                    int readInt = dis.readInt();
                    String username = dis.readUTF();
                    User user = new User(readInt, username);
    
                    // dos会自动帮助关闭baos的流，手动关闭也不会出错
                    dos.close();
                    s.close();
                    dis.close();
    
                    return user;
                }
            };
    
    
            // 生成动态代理对象
            // 三个参数：1、ClassLoader；2、interfaces；3、handler
            Object o = Proxy.newProxyInstance(IUserService.class.getClassLoader(), new Class[]{IUserService.class}, handler);
    
            return (IUserService) o;
        }
    
    
    }
    ```

- Client

    ```java
    public class Client {
    
        public static void main(String[] args) {
            IUserService service = Stub.getStub();
            System.out.println(service.findUserById(123));
        }
    }
    ```

> 这里只是为了过渡到动态代理的模式，用了Proxy和InvocationHandler。
>
> 实际使用中跟动态代理的关系不大。因为不管调用什么方法，都执行findUserById





# 4 动态代理发挥作用



- Stub

    ```java
    public class Stub {
    
        public static IUserService getStub() {
            InvocationHandler handler = new InvocationHandler() {
    
                /**
                 * 定义处理器，方法实际执行者
                 * @param proxy 被代理的对象
                 * @param method 被代理对象的实际执行方法
                 * @param args 方法的参数
                 * @return
                 * @throws Throwable
                 */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    Socket s = new Socket("localhost", 8888);
    
                    ObjectOutputStream oos = new ObjectOutputStream(s.getOutputStream());
    
                    String methodName = method.getName();
                    // 泛型支持
                    Class<?>[] parameterTypes = method.getParameterTypes();
                    //1.写入方法名
                    oos.writeUTF(methodName);
                    //2.写入方法参数类型 基于1和2，这样就可以在所有重载函数中精确找到方法
                    oos.writeObject(parameterTypes);
                    //3.写入具体参数
                    oos.writeObject(args);
                    oos.flush();
    
                    // 接受来自服务端的信息
                    DataInputStream dis = new DataInputStream(s.getInputStream());
    
                    int readInt = dis.readInt();
                    String username = dis.readUTF();
                    User user = new User(readInt, username);
    
                    // dos会自动帮助关闭baos的流，手动关闭也不会出错
                    oos.close();
                    s.close();
                    dis.close();
    
                    return user;
                }
            };
    
    
            // 生成动态代理对象
            Object o = Proxy.newProxyInstance(IUserService.class.getClassLoader(), new Class[]{IUserService.class}, handler);
    
            return (IUserService) o;
        }
    
    
    }
    ```

- Client

    ```java
    public class Client {
    
        public static void main(String[] args) {
            IUserService userService = Stub.getStub();
            User user = userService.findUserById(23);
            System.out.println(user);
        }
    }
    ```

- Server

    ```java
    public class Server {
    
        private static boolean running = true;
    
        public static void main(String[] args) {
            try {
                ServerSocket ss = new ServerSocket(8888);
                System.out.println("===============> 开始监听");
                while (running) {
                    Socket s = ss.accept();
                    InetAddress inetAddress = s.getInetAddress();
                    System.out.println("接收到来自ip = " + inetAddress + "的请求！");
                    process(s);
                    s.close();
                }
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            } catch (NoSuchMethodException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } finally {
                System.out.println("已经返回！");
            }
        }
    
        public static void process(Socket socket) throws IOException, ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
            InputStream inputStream = socket.getInputStream();
            OutputStream outputStream = socket.getOutputStream();
            // 换成了ObjectInputStream
            ObjectInputStream ois = new ObjectInputStream(inputStream);
            DataOutputStream dataOutputStream = new DataOutputStream(outputStream);
    
            // 读入方法名
            String methodName = ois.readUTF();
            // 读入参数类型
            Class[] parameterTypes = (Class[]) ois.readObject();
            // 读入参数
            Object[] args = (Object[]) ois.readObject();
    
            UserServiceImpl userService = new UserServiceImpl();
            // 根据方法名 & 方法参数 精确获取方法
            Method method = userService.getClass().getMethod(methodName, parameterTypes);
            // 调用方法获得数据
            User user = (User) method.invoke(userService, args);
    
            dataOutputStream.writeInt(user.getId());
            dataOutputStream.writeUTF(user.getUsername());
            dataOutputStream.flush();
    
            ois.close();
            dataOutputStream.close();
    
        }
    }
    ```

> 在这个版本中，代码能够支持IUserService接口下的所有方法的反射调用。



# 5 修改原本返回id和username，改为Object

返回值用Object封装，支持任意类型。



- Stub修改部分

    ```java
    // 接受来自服务端的信息
    ObjectInputStream dis = new ObjectInputStream(s.getInputStream());
    User user = (User) dis.readObject();
    ```

- Server.process()

    ```java
    public static void process(Socket socket) throws IOException, ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        InputStream inputStream = socket.getInputStream();
        OutputStream outputStream = socket.getOutputStream();
        ObjectInputStream ois = new ObjectInputStream(inputStream);
        ObjectOutputStream oos = new ObjectOutputStream(outputStream);
    
        // 读入方法名
        String methodName = ois.readUTF();
        // 读入参数类型
        Class[] parameterTypes = (Class[]) ois.readObject();
        // 读入参数
        Object[] args = (Object[]) ois.readObject();
    
        UserServiceImpl userService = new UserServiceImpl();
        // 根据方法名 & 方法参数 精确获取方法
        Method method = userService.getClass().getMethod(methodName, parameterTypes);
        // 调用方法获得数据
        User user = (User) method.invoke(userService, args);
    
        // 写入的是对象 ：修改处
        oos.writeObject(user);
        oos.flush();
    
        ois.close();
        oos.close();
    
    }
    ```

- pojo

    ```java
    /**
    * 要要网络通信中传递Object，就必须实现Serializable
    */
    public class User implements Serializable {
    
        private Integer id;
        private String username;
    
    }
    ```

> 至此，依然存在问题：
>
> 返回的对象必须是User, 不能是其他的，也不能是List<User>
>
> 而且也只能用于User对象，我们需要支持更多的对象。



# 6 支持返回任意类型的对象



之前我们的Stub返回一个IUserService对象。现在我们要返回Object。

- Stub

    ```java
    package com.kicc.rpc06;
    
    import com.kicc.common.pojo.User;
    import com.kicc.common.service.IUserService;
    
    import java.io.ObjectInputStream;
    import java.io.ObjectOutputStream;
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;
    import java.net.Socket;
    import java.util.List;
    
    /**
     * @author Kicc
     * @date 20/7/9 下午 4:12
     */
    public class Stub {
    
        public static Object getStub(final Class clazz) {
            InvocationHandler handler = new InvocationHandler() {
    
                /**
                 * 定义处理器，方法实际执行者
                 * @param proxy 被代理的对象
                 * @param method 被代理对象的实际执行方法
                 * @param args 方法的参数
                 * @return
                 * @throws Throwable
                 */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    Socket s = new Socket("localhost", 8888);
    
                    ObjectOutputStream oos = new ObjectOutputStream(s.getOutputStream());
    
                    String clazzName = clazz.getName();
                    String methodName = method.getName();
                    Class<?>[] parameterTypes = method.getParameterTypes();
                    Class<?> returnType = method.getReturnType();
    
                    //0.写入类型
                    oos.writeUTF(clazzName);
                    //1.写入方法名
                    oos.writeUTF(methodName);
                    //2.写入方法参数类型 基于1和2，这样就可以在所有重载函数中精确找到方法
                    oos.writeObject(parameterTypes);
                    //3.写入具体参数
                    oos.writeObject(args);
                    //4.写入返回值类型
    //                oos.writeObject(returnType);
                    oos.flush();
    
                    // 接受来自服务端的信息
                    ObjectInputStream ois = new ObjectInputStream(s.getInputStream());
                    Object o = ois.readObject();
    
                    // dos会自动帮助关闭baos的流，手动关闭也不会出错
                    oos.close();
                    s.close();
                    ois.close();
    
                    return o;
                }
            };
    
    
            // 生成动态代理对象,通过clazz生成
            Object o = Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{clazz} , handler);
    
            return o;
        }
    
    
    }
    ```

    - 修改1：增加了getStub的参数 Class
    - 修改2：修改返回值类型为Object
    - 增加了类型的获取 String clazzName = clazz.getName();
    - 动态代理对象也生成了Object，参数都与clazz相关

- Client

    ```java
    public class Client {
    
        public static void main(String[] args) {
            IUserService userService = (IUserService) Stub.getStub(IUserService.class);
            User user = userService.findUserById(23);
            System.out.println(user);
    
    
        }
    }
    ```

- Server

    ```java
        private static void process(Socket socket) throws IOException, ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException, InstantiationException {
            InputStream inputStream = socket.getInputStream();
            OutputStream outputStream = socket.getOutputStream();
            ObjectInputStream ois = new ObjectInputStream(inputStream);
    
    
    
            // 读入 class
            String clazzname = ois.readUTF();
            System.out.println("类型为 "+ clazzname);
            // 读入方法名
            String methodName = ois.readUTF();
            // 读入参数类型
            Class[] parameterTypes = (Class[]) ois.readObject();
            // 读入参数
            Object[] args = (Object[]) ois.readObject();
    
            Class clazz = null;
    
            // 从服务注册表中找到具体类 用Spring注入
            clazz = UserServiceImpl.class;
    
            // 根据方法名 & 方法参数 精确获取方法
            // 相当于 new UserServiceImpl().findUserById(id);
            Method method = clazz.getMethod(methodName, parameterTypes);
    
            // 调用方法获得数据
            Object o =  method.invoke(clazz.newInstance(), args);
    
            ObjectOutputStream oos = new ObjectOutputStream(outputStream);
            oos.writeObject(o);
            oos.flush();
    
            ois.close();
            oos.close();
    
        }
    }
    ```



> 解决：List<Object>这样的返回类型已支持。因为Object也是List的父类。
>
> Server中的对象用Spring注入就行。



# 7 对6举例说明

现在，除了User类、IUserService接口、UserServiceImpl实现类之外，我们又有了Product相关的类。



Stub 和 Server 几乎不需要变动，（Server需要找到对应的实现，可以通过clazzname找，Spring注入也可）。就能在Client中直接调用!

```java
public class Client {

    public static void main(String[] args) {
        IProductService productService = (IProductService) Stub.getStub(IProductService.class);

        List<User> allProducts = productService.findAllProducts();

        System.out.println(allProducts);


    }
}
```



# 8 序列化实现方式

之前讲到，网络通信最终是传递的是序列化的二进制数据。以上都是基于Java原生的序列化技术。效率不高。



**常见的RPC序列化框架**：

1. java.io.Serializable
2. Hessian
3. google protobuf
4. faceboot Thrift
5. kyro
6. fst
7. json 序列化框架：对象先转化成json，再由框架转成二进制
    1. Jackson
    2. google Gson
    3. Ali FastJson
8. xmlrpc (xstream)
9. ...



基于Hessian举例：

> Hessian是一种二进制Web服务协议，它使Web服务可以使用，而无需大型框架，也无需学习新的协议集。由于它是二进制协议，因此非常适合发送二进制数据，而无需使用附件扩展协议。Hessian由Caucho Technology，Inc.开发

```java
public class HelloHessian {
    public static void main(String[] args) throws IOException {
        User kicc = new User(1, "大哥大");
        byte[] bytes = serialize(kicc);
        System.out.println(bytes.length);
    }

    public static byte[] serialize(Object o) throws IOException {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        Hessian2Output hessian2Output = new Hessian2Output(byteArrayOutputStream);
        // 核心写入
        hessian2Output.writeObject(o);
        hessian2Output.flush();
        byte[] bytes = byteArrayOutputStream.toByteArray();
        byteArrayOutputStream.close();
        hessian2Output.close();
        return bytes;
    }

    public static Object deserialize(byte[] bytes) throws IOException {
        ByteArrayInputStream inputStream = new ByteArrayInputStream(bytes);
        Hessian2Input hessian2Input = new Hessian2Input(inputStream);
        Object o = hessian2Input.readObject();
        inputStream.close();
        hessian2Input.close();
        return o;
    }

}
```

- 序列化
    - 创建字节输出流
    - 创建Hession输出流
    - 写入对象
    - 字节输出流导入字节数组 （为了return 字节数组而写，不写也可以）



# 9 JDK的序列化和Hessian的比较



```java
public static void main(String[] args) throws IOException {
    User kicc = new User(1, "大哥大");
    byte[] bytes = hessianSerialize(kicc);
    System.out.println(bytes.length);

    byte[] bytes1 = jdkSerialize(kicc);
    System.out.println(bytes1.length);
}

52
194
```

可以看到，JDK的序列化字节数组的长度为194，而Hessian序列化的字节数组长度为194。足足有4倍差距。



把JDK原生的序列化替换只需要：

```java
// Hessian方式 输出流
Hessian2Output oos = new Hessian2Output(s.getOutputStream());
// 原生 输出流
ObjectOutputStream oos = new ObjectOutputStream(s.getOutputStream());
// Hessian 输入流
Hessian2Input ois = new Hessian2Input(s.getInputStream());
// JDK 输入流
ObjectInputStream ois = new ObjectInputStream(s.getInputStream());
```

接受一个Stream，包装成Hessian。





# 10 消息编码和解码

## 一、消息数据结构 

在invoke方法中封装通信细节，信的第一步就是要确定客户端和服务端相互通信的消息结构。客户端的请求消息结构一般需要包括以下内容：

- 接口名称 （IUserService）
- 方法名 Method
- 参数类型 、 参数值
- 超时时间
- requestID

服务器端的数据结构：

1. 返回值
2. 状态code
3. requestID 



## 二、为什么需要requestID ?

简单的说，请求是异步的，同一个服务端可能收到来自同一个客户端的多个不同请求。requestID用来唯一标识，避免返回给错误的请求。

具体解决：（基于netty）

1. client线程每次通过socket调用一次远程接口前，生成一个唯一的ID，即requestID（requestID必需保证在一个Socket连接里面是唯一的），一般常常使用AtomicLong从0开始累计数字生成唯一ID；
2. 将处理结果的回调对象callback，存放到全局ConcurrentHashMap里面put(requestID, callback)；
3. 当线程调用channel.writeAndFlush()发送消息后，紧接着执行callback的get()方法试图获取远程返回的结果。在get()内部，则使用synchronized获取回调对象callback的锁，再先检测是否已经获取到结果，如果没有，然后调用callback的wait()方法，释放callback上的锁，让当前线程处于等待状态。
4. 服务端接收到请求并处理后，将response结果（此结果中包含了前面的requestID）发送给客户端，客户端socket连接上专门监听消息的线程收到消息，分析结果，取到requestID，再从前面的ConcurrentHashMap里面get(requestID)，从而找到callback对象，再用synchronized获取callback上的锁，将方法调用结果设置到callback对象里，再调用callback.notifyAll()唤醒前面处于等待状态的线程。



# 11 RPC 小结

RPC中，除了序列化这一部分以外，另外一部分很重要的是网络的传输协议。

> 不仅序列化框架可以选择，网络的传输协议也可以选择。

常见的用TCP、IP协议进行二进制数据的传输，也可以用Http、Mail、UDP等。

- http+json，Restful
- http1.0 只支持文本
- http2.0 (gRPC) 支持二进制
- TCP
    - 同步、异步、阻塞、非阻塞
- WebService



RPC只是一种通信方式，实现方式可以用CORBA（古老）、TCP/UDP二进制传输（最古老、最底层）、各种WebService（SOA、SOAP、RDDI..)、RestFu Api、RMI（Java的RPC自带实现）.





# 12 服务的发布

如何让别人使用我们的服务呢？

- 人肉告知
- 自动告知



**人肉告知方式：**

> 人肉告知的方式：如果你发现你的服务一台机器不够，要再添加一台，这个时候就要告诉调用者我现在有两个ip了，你们要轮询调用来实现负载均衡；调用者咬咬牙改了，结果某天一台机器挂了，调用者发现服务有一半不可用，他又只能手动修改代码来删除挂掉那台机器的ip。现实生产环境当然不会使用人肉方式。



**自动告知方式**：zookeeper



![img](https://images2015.cnblogs.com/blog/522490/201510/522490-20151003183747543-2138843838.png)





> zookeeper充当一个服务注册表（Service Registry），让多个`服务提供者`形成一个集群，让`服务消费者`通过服务注册表获取具体的服务访问地址（ip+端口）去访问具体的服务提供者。

具体来说，zookeeper就是个分布式文件系统，每当一个服务提供者部署后都要将自己的服务注册到zookeeper的某一路径上: /{service}/{version}/{ip:port}, 比如我们的HelloWorldService部署到两台机器，那么zookeeper上就会创建两条目录：分别为/HelloWorldService/1.0.0/100.19.20.01:16888  /HelloWorldService/1.0.0/100.19.20.02:16888。



zookeeper提供了“心跳检测”功能，它会定时向各个服务提供者发送一个请求（实际上建立的是一个 Socket 长连接），如果长期没有响应，服务中心就认为该服务提供者已经“挂了”，并将其剔除，比如100.19.20.02这台机器如果宕机了，那么zookeeper上的路径就会只剩/HelloWorldService/1.0.0/100.19.20.01:16888。

　　

服务消费者会去监听相应路径（/HelloWorldService/1.0.0），一旦路径上的数据有任务变化（增加或减少），zookeeper都会通知服务消费方服务提供者地址列表已经发生改变，从而进行更新。

　　

更为重要的是zookeeper与生俱来的容错容灾能力（比如leader选举），可以确保服务注册表的高可用性。