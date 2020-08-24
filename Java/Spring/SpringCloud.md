# Dubbo面试题 

喜欢问RPC框架

## 基础问题

### 为什么要用？

在微服务架构出来之前，大多数的应用都是一种 `all in one` 的形式呈现的。这种方式的架构在 服务、模块量少的情况下表现的很好。

但是，随着服务量越来越多、服务和服务之间的调用、依赖关系也变的更加复杂 ==> **诞生** ==> SOA (面向服务) 架构体系。于是出现：

- 服务提供
- 服务调用
- 连接处理
- 通信协议
- 序列化方式
- 服务发现
- 服务路由
- ...

Dubbo就是对这么多的行为进行一个封装，进行综合治理。







### 是什么？

根据官网的描述，Dubbo是一个Java开发的，高性能的RPC框架。

提供了服务自动注册、自动发现等功能

可以和Spring无缝集成



### 使用场景？

- RPC调用，调用远程的服务就像本地方法调用一样
- 服务自动注册与发现：不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者。



### 可以解决的问题

微服务的四个核心问题

- 服务多，客户端如何访问？
- 服务多，服务之间如何通信？
- 服务多，如何治理？
- 服务如果挂了，如何处理？



为此，无论是SpringCloud Netflix \ Dubbo+Zookeeper \ SpringCloud Alibaba，都是针对性地去解决这四个问题。

> 客户端如何访问？

有api网关

- zuul组件



> 如何通信？

Http 、RPC

- Dubbo



> 如何治理？

服务的注册与发现

- Zookeeper
- Erueka



> 意外情况的应对

熔断机制

- Hystrix





### 带来的其他问题

- 数据一致性问题
- 网络通信问题



### 核心功能是什么

- Remoting：网络通信框架
- Cluster：服务框架，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
- Register：服务注册。都是自动的。很方便的增加或者删除一个或多个机器。



### 核心组件

![在这里插入图片描述](SpringCloud.assets/1717841c2bd87ef0)



**Provider**：暴露服务的服务提供方

**Consumer**：调用远程服务消费方

**Registry**：服务注册与发现注册中心

**Monitor**：监控中心和访问调用统计

**Container**：服务运行容器



### Dubbo 服务器注册与发现的流程？

- 服务容器Container负责启动，加载，运行服务提供者。
- 服务提供者Provider在启动时，向注册中心注册自己提供的服务。
- 服务消费者Consumer在启动时，向注册中心订阅自己所需的服务。
- 注册中心Registry返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
- 服务消费者Consumer，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 服务消费者Consumer和提供者Provider，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。





## （重点）负载均衡策略

**啥是负载均衡？**

在单次的服务调用上，选择一台合适的服务器进行服务调用。并且设置不止一台服务器提供服务。消费者可以通过策略选择最合适的服务器进行服务的调用。缓解一台服务器的压力。



**与之相关的还有两个概念**

1. 负载均衡
2. 集群容错
3. 服务路由

> 用一个例子来说明这三个概念

有一个Dubbo的用户服务，在北京部署了10个，在上海部署了20个。一个杭州的服务消费方发起了一次调用，然后发生了以下的事情:

1. 根据配置的路由规则，如果杭州发起的调用，会路由到比较近的上海的20个 Provider。**（服务路由）**
2. 根据配置的随机负载均衡策略，在20个 Provider 中随机选择了一个来调用，假设随机到了第7个 Provider。**（负载均衡）**
3. 结果调用第7个 Provider 失败了。
4. 根据配置的Failover集群容错模式，重试其他服务器。**（集群容错）**
5. 重试了第13个 Provider，调用成功。
6. 

上面的第1，2，4步骤就分别对应了路由，负载均衡和集群容错。Dubbo中，**先通过路由，从多个 Provider 中按照路由规则**，选出一个子集。再根据负载均衡从子集中选出一个 Provider 进行本次调用。如果调用失败了，根据**集群容错策略**，进行重试或定时重发或快速失败等。 可以看到Dubbo中的路由，负载均衡和集群容错发生在**一次RPC调用**的**不同阶段**。最先是路由，然后是负载均衡，最后是集群容错。 本文档只讨论负载均衡，路由和集群容错在其他的文档中进行说明。





### Dubbo提供了哪些策略？

- **Random LoadBalance:** **随机选取**提供者策略，**有利于动态调整提供者权重**。截面碰撞率高，调用次数越多，分布越均匀。
    - 不是说完全的随机，每个provider是有一个权重的
    - 比如说性能好一点的机器，权重就大一点

- **RoundRobin LoadBalance:** 轮询负载均衡，平均分布，但是存在请求累积的问题。
    - 比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上，导致整个系统变慢。

- **LeastActive LoadBalance:** 最少活跃调用策略，解决慢提供者接收更少的请求。
    - 让执行服务慢的机器接收更少的请求，执行快的机器多执行点请求

- **ConstantHash LoadBalance:** 一致性 Hash 策略，使**相同参数请求**总是发到同一提供者，一台机器宕机，可以基于虚拟节点，分摊至其他提供者，避免引起提供者的剧烈变动。
    - 一致性Hash算法可以和缓存机制配合起来使用。比如有一个服务getUserInfo(String userId)。设置了Hash算法后，相同的userId的调用，都会发送到同一个 Provider。这个 Provider 上可以把**用户数据在内存中进行缓存**，减少访问数据库或分布式缓存的次数。如果业务上允许这部分数据有一段时间的不一致，可以考虑这种做法。减少对数据库，缓存等中间件的依赖和访问次数，同时减少了网络IO操作，提高系统性能。



## 配置负载均衡

如果不指定负载均衡，默认使用随机负载均衡。我们也可以根据自己的需要，显式指定一个负载均衡。 可以在多个地方类来配置负载均衡，比如 Provider 端，Consumer端，服务级别，方法级别等。

### 服务端服务级别

```xml
<dubbo:service interface="..." loadbalance="roundrobin" />
```

该服务的所有方法都使用roundrobin负载均衡。

### 客户端服务级别

```xml
<dubbo:reference interface="..." loadbalance="roundrobin" />
```

该服务的所有方法都使用roundrobin负载均衡。

### 服务端方法级别

```xml
<dubbo:service interface="...">
    <dubbo:method name="hello" loadbalance="roundrobin"/>
</dubbo:service>
```

只有该服务的hello方法使用roundrobin负载均衡。

### 客户端方法级别

```xml
<dubbo:reference interface="...">
    <dubbo:method name="hello" loadbalance="roundrobin"/>
</dubbo:reference>
```

只有该服务的hello方法使用roundrobin负载均衡。

和Dubbo其他的配置类似，多个配置是有覆盖关系的：

1. 方法级优先，接口级次之，全局配置再次之。
2. 如果级别一样，则**消费方优先**，提供方次之。

所以，上面4种配置的优先级是:

1. 客户端方法级别配置。
2. 客户端接口级别配置。
3. 服务端方法级别配置。
4. 服务端接口级别配置。





## 集群容错方案

> 定义

在一个集群中有许多的服务器，当客户端发起一个服务调用请求后，会通过负载均衡策略选一个provider。但是在线上环境中存在着各种各样的情况，会使我们某次的服务调用不成功。这个时候，集群就需要有一种容错机制，找到下一个可以调用的provider。





### 内置容错策略

Dubbo主要内置了如下几种策略：

- Failover(失败自动切换)
    - 当主服务器不行，自动切换的备用服务器。默认是最多重试**两次**。选一个可行的备用地址。
- Failsafe(失败安全)
    - 出现异常时，直接忽略。通常用于写入审计日志等操作。
- Failfast(快速失败)
    - 只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
- Failback(失败自动恢复)
- Forking(并行调用)
- Broadcast(广播调用)



### 具体配置 （面试应该不需要问）



#### Failover

以XML方式为例，具体配置方法如下：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="failover" retries="2" />
```

服务提供方，方法级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService"cluster="failover">
     <dubbo:method name="sayHello" retries="2" />
 </dubbo:reference>
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="failover" retries="1"/>
```

服务调用方，方法级配置：

```xml
 <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="failover">
     <dubbo:method name="sayHello" retries="3" />
 </dubbo:reference>
```

Failover可以自动对失败进行重试，对调用者屏蔽了失败的细节，但是Failover策略也会带来一些副作用：

- 重试会额外增加一下开销，例如增加资源的使用，在高负载系统下，额外的重试可能让系统雪上加霜。
- 重试会增加调用的响应时间。
- 某些情况下，重试甚至会造成资源的浪费。考虑一个调用场景，A->B->C，如果A处设置了超时100ms，再B->C的第一次调用完成时已经超过了100ms，但很不幸B->C失败，这时候会进行重试，但其实这时候重试已经没有意义，因此在A看来这次调用已经超时，A可能已经开始执行其他逻辑。



#### Failsafe(失败安全)

具体配置方法：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="failsafe" />
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="failsafe"/>
```

其中服务调用方配置优先于服务提供方配置。



#### Failfast(快速失败)

具体配置方法：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="failfast" />
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="failfast"/>
```

其中服务调用方配置优先于服务提供方配置。



#### Failback(失败自动恢复)

具体配置方法：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="failsafe" />
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="failsafe"/>
```

其中服务调用方配置优先于服务提供方配置。

按照目前的实现，Failback策略还有一些局限，例如内存中的失败调用列表没有上限，可能导致堆积，异步重试的执行间隔无法调整，默认是5秒。



#### Forking(并行调用)

具体配置方法：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="forking" />
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="forking"/>
```

其中服务调用方配置优先于服务提供方配置。



#### Broadcast(广播调用)

具体配置方法：

服务提供方，服务级配置

```xml
<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" cluster="broadcast" />
```

服务调用方，服务级配置

```xml
<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" cluster="broadcast"/>
```

其中服务调用方配置优先于服务提供方配置。