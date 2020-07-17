# 1 NoSQL 概述



## 1.1 为什么使用？

> 1. 单机模型

最早的单机服务器上，我们只需要使用单个的数据库就够使用了。毕竟用户的访问量没有那么大。



> 2. Memcached（缓存）+ MySQL + 垂直拆分

- 网站80%的请求都是读，每次都要取数据库中查询的话效率太低了。

- 为了减少服务器的压力，我们就可以使用缓存来保证效率。

    ![image-20200628192246257](Redis.assets/image-20200628192246257.png)

    

- 发展过程：优化数据结构和索引 ==》文件缓存（IO）==》Memcached

缓存技术主要是来解决**读**的问题。



> 3. 水平拆分 分库分表 MySQL集群



> 为什么要用？

- 数据量增长的速度非常快，单单用关系型数据库很难处理。
- MySQL来存储些比较大的文件，博客，图片时，效率不高。需要依赖另一种专门处理的数据库。



## 1.2 什么是NoSQL

很多的数据类型：用户的个人信息，社交网络，地理信息等。这些数据类型的存储不需要一个固定的格式。不需要多余的操作就可以横向扩展的。 就像 Map <String, Object>

> NoSQL 特点

​	**解耦！**

1. 方便扩展：数据之间没有关系，很容易扩展。

2. 大数据量高性能：Redis 一秒写8W，读取11W。NoSQL的缓存记录级，是一种细粒度的缓存。

3. 数据类型是多样性的：不需要事先设计数据库。随去随用。如果是数据量大的表，一开始很难设计好。

4. 传统RDBMS和NoSQL

    ```yaml
    传统RDBMS
    - 结构化组织
    - SQL 语言
    - 数据和关系都存在单独的表中
    - 操作，数据定义语言
    - 严格的一致性
    - 基础的事务
    ```

    ```
    NoSQL
    - 不仅仅是数据
    - 没有固定的查询语言
    - 从多种存储方式：键值对存储，列存储，文档存储，图形数据库
    - 最终一致性，保证最终一致就可以
    - CAP定理 和 BASE
    - 高性能，高可用，高可扩展
    ```

    

> 了解：3V+3高

- 大数据的3V：用来描述问题；
    - 海量Volume
    - 多样Variety
    - 实时Velocity
- 3高需求：
    - 高并发
    - 高可扩 （随时水平拆分，+服务器）
    - 高性能

最佳实践：NoSQL + RDBMS。





# 架构演进（阿里）



**一、 商品的各种数据信息**

```bash
# 1、商品的基本信息
	名称、价格、商家信息
	关系型数据库就可以解决。 MySQL / Oracle
	淘宝内部的MySQL不是大家用的MySQL
	
	
# 2、商品的描述、评论（文字比较多）
	文档型数据库：MongoDB

# 3、图片
	分布式文件系统 FastDFS
	- 淘宝自己的 TFS
	- Hadoop   HDFS
	- 阿里云    oss
	
# 4、商品的关键字（搜索）
	- 搜索引擎 solr elasticsearch
	
# 5、商品热门的波段信息
	- 内存数据库
	- Redis Tair、Memcache
	
# 6、商品的交易，外部的支付接口
	- 三方应用
```



**二、大型互联网中的问题**

- 数据类型太多了
- 数据源繁多，经常重构
- 数据要改造的话，大面积改造？

解决办法：统一数据服务层框架缓存模块 UDSL

> 在网站APP与各种数据库相关底层中间加上一层统一的数据处理层。屏蔽底层细节。方便程序员编程。
>
> 程序员不需要去了解每一个底层数据库或者搜索引擎的具体细节。只要用一个统一的接口操作一切就可以。



# NoSQL的四种类型



**KV键值对**：

- 新浪：Redis
- 美团：Redis+Tair
- 阿里、百度：Redis——memcache



**文档型数据库（bson格式）**：

- MongoDB（一般必须要掌握）
    - MongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档。
    - MongoDB是一个介于关系型数据库和NoSQL之间的一种产品。MongoDB是非关系型数据库中功能最丰富，最像关系型数据的。
- CouchDB



**列存储的数据库**：

- HBase
- 分布式文件系统



**图数据库**：

- neo4j





# Redis 入门

## 概述

> Redis = Remote Dictionary Server 远程字典服务

Redis 是 开源、C语言编写、支持网络、可基于内存、可持久化、K-V数据库，提供多种语言API。 

Redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。



> Redis 能干嘛？

1、内存存储、持久化。持久化就聊 rdb和aof。

2、效率高，可以用于高速缓存。

3、发布订阅系统

4、地图信息分析

5、计时器、计数器（微信浏览量）

6、.....



> 特性

1、多样化数据类型

2、持久化

3、集群

4、事务



> 





## Linux 安装

1. 下载redis-6.0.x.tar.gz

2. ```bash
    tar -zxvf redis-.tar.gz
    cd redis-.tar.gz
    ```

3. ```bash
    # make
    # 安装在/usr/local/bin
    make
    make install
    ```

查看：![image-20200715230923500](Redis.assets/image-20200715230923500.png)

4. 拷贝解压包下的redis.conf到 /usr/local/bin/redisconfig

> 我们以后就修改这个配置文件进行启动。

5. redis不是默认后台启动的，修改配置文件。

    ![image-20200715231357151](Redis.assets/image-20200715231357151.png)



6. 启动redis服务：通过配置文件启动 redis-server

![image-20200715231530305](Redis.assets/image-20200715231530305.png)

7. 测试连接：使用redis-cli 指定端口号 （-h 指定host ip）

    ![image-20200715231759249](Redis.assets/image-20200715231759249.png)

8. 退出redis服务：server & cli

    ![image-20200715232000780](Redis.assets/image-20200715232000780.png)

    9.后续使用单机多port的集群方式。



## 性能测试

**redis-benchmark**

redis 性能测试工具可选参数如下所示：

| 序号 | 选项      | 描述                                       | 默认值    |
| :--- | :-------- | :----------------------------------------- | :-------- |
| 1    | **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**    | 指定服务器端口                             | 6379      |
| 3    | **-s**    | 指定服务器 socket                          |           |
| 4    | **-c**    | 指定并发连接数                             | 50        |
| 5    | **-n**    | 指定请求数                                 | 10000     |
| 6    | **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 3         |
| 7    | **-k**    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**    | 生成循环，永久执行测试                     |           |
| 13   | **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

```bash
# 测试 100个并发 每个并发100000条请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

![image-20200716102642094](Redis.assets/image-20200716102642094.png)







## 基础的知识

>  redis默认有16个数据库。在redis.conf中可以设置个数。15个初始化都是空的。

默认使用的是第0个数据库，可以使用  ==select n== 切换数据库 .



清空当前数据库 `flushdb`

```bash
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> keys *
(empty list or set)

```

情况所有数据库 `flushall`



> Redis 是单线程的！

Redis的速度是很快的，官方表示，Redis是基于内存操作的，CPU不是Redis的性能瓶颈。Redis是瓶颈是根据机器的内存和网络带宽，既然可以使用单线程来实现，就使用单线程了。

Reids是C语言写的，官方数据 100000+QPS，完全不比同样使用 Key-Value的Memcache差。



**为什么单线程还这么快？**

1、误区1：高性能的服务器一定是多线程的

2、误区2：多线程一定比单线程效率高。（上下文的切换消耗资源的）



核心：Redis是将全部的数据放在内存中的。所以用单线程去操作效率就高。用了多线程的上下文切换还不如单线程。





# 五大数据类型

> 介绍

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作==数据库==、==缓存==和==消息中间件MQ==。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

### Redis-Key

下表给出了与 Redis 键相关的基本命令：

| 序号   | 命令及描述                                                   |
| :----- | :----------------------------------------------------------- |
| ==1==  | [DEL key](https://www.runoob.com/redis/keys-del.html) 该命令用于在 key 存在时删除 key。 |
| 2      | [DUMP key](https://www.runoob.com/redis/keys-dump.html) 序列化给定 key ，并返回被序列化的值。 |
| ==3==  | [EXISTS key](https://www.runoob.com/redis/keys-exists.html) 检查给定 key 是否存在。 |
| ==4==  | [EXPIRE key](https://www.runoob.com/redis/keys-expire.html) seconds 为给定 key 设置过期时间，以秒计。 |
| 5      | [EXPIREAT key timestamp](https://www.runoob.com/redis/keys-expireat.html) EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| 6      | [PEXPIRE key milliseconds](https://www.runoob.com/redis/keys-pexpire.html) 设置 key 的过期时间以毫秒计。 |
| 7      | [PEXPIREAT key milliseconds-timestamp](https://www.runoob.com/redis/keys-pexpireat.html) 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 |
| 8      | [KEYS pattern](https://www.runoob.com/redis/keys-keys.html) 查找所有符合给定模式( pattern)的 key 。 |
| ==9==  | [MOVE key db](https://www.runoob.com/redis/keys-move.html) 将当前数据库的 key 移动到给定的数据库 db 当中。 |
| 10     | [PERSIST key](https://www.runoob.com/redis/keys-persist.html) 移除 key 的过期时间，key 将持久保持。 |
| 11     | [PTTL key](https://www.runoob.com/redis/keys-pttl.html) 以毫秒为单位返回 key 的剩余的过期时间。 |
| ==12== | [TTL key](https://www.runoob.com/redis/keys-ttl.html) 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| 13     | [RANDOMKEY](https://www.runoob.com/redis/keys-randomkey.html) 从当前数据库中随机返回一个 key 。 |
| 14     | [RENAME key newkey](https://www.runoob.com/redis/keys-rename.html) 修改 key 的名称 |
| 15     | [RENAMENX key newkey](https://www.runoob.com/redis/keys-renamenx.html) 仅当 newkey 不存在时，将 key 改名为 newkey 。 |
| 16     | [SCAN cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/keys-scan.html) 迭代数据库中的数据库键。 |
| ==17== | [TYPE key](https://www.runoob.com/redis/keys-type.html) 返回 key 所储存的值的类型。 |



### String 

```bash
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
1) "age"
127.0.0.1:6379[1]> type name
none
127.0.0.1:6379[1]> type age
string
127.0.0.1:6379[1]> keys *
1) "age"
127.0.0.1:6379[1]> type age
string
127.0.0.1:6379[1]> APPEND age 1     # append string
(integer) 3
127.0.0.1:6379[1]> type age
string
127.0.0.1:6379[1]> get age
"251"
127.0.0.1:6379[1]> STRLEN age
(integer) 3
127.0.0.1:6379[1]> APPEND age ",岁"   # 中文占了三个长度
(integer) 7
127.0.0.1:6379[1]> get age
"251,\xe5\xb2\x81"
127.0.0.1:6379[1]>

#################################################
# 自增 自减 步长
127.0.0.1:6379[1]> set views 0
OK
127.0.0.1:6379[1]> get views
"0"
127.0.0.1:6379[1]> INCR views   # i++
(integer) 1
127.0.0.1:6379[1]> DECR views	# i--
(integer) 0
127.0.0.1:6379[1]> type views
string
127.0.0.1:6379[1]> INCRBY views 1-
(error) ERR value is not an integer or out of range
127.0.0.1:6379[1]> INCRBY views 10  # i+n
(integer) 10
127.0.0.1:6379[1]> DECRBY views 6	# i-n
(integer) 4
127.0.0.1:6379[1]>
#################################################
# 字符串范围
127.0.0.1:6379[1]> set key1 "What's your name?"
OK
127.0.0.1:6379[1]> GETRANGE key1 0 4    # 闭区间截取字符串
"What'"
127.0.0.1:6379[1]> GETRANGE key1 0 -1   # 获取全部
"What's your name?"
127.0.0.1:6379[1]>


#################################################
# “存”对象，实际上还是String，但是长的像json
127.0.0.1:6379[1]> set user:1 {name:kicc,age:22}
OK
127.0.0.1:6379[1]> DEL user:1:name
(integer) 1
127.0.0.1:6379[1]> DEL user:1:age
(integer) 1
127.0.0.1:6379[1]> mset user:1:name zhangsan user:1:age 11
OK
127.0.0.1:6379[1]> keys *
1) "user:1:name"
2) "user:1"
3) "user:1:age"
127.0.0.1:6379[1]> mget user:1:name user:1:age
1) "zhangsan"
2) "11"
127.0.0.1:6379[1]>

# 以Bilibili用户的关注数举例，我们可以在redis中这样存储
mset uid:xxxxxxx:follow 0 # 初始关注数为0
incr uid:xxxxxxx:follow  # 有新的关注
decr uid:xxxxxxx:follow  # 有人取消关注
#################################################
```



下表列出了常用的 redis 字符串命令：

| 序号   | 命令及描述                                                   |
| :----- | :----------------------------------------------------------- |
| 1      | [SET key value](https://www.runoob.com/redis/strings-set.html) 设置指定 key 的值 |
| 2      | [GET key](https://www.runoob.com/redis/strings-get.html) 获取指定 key 的值。 |
| ==3==  | [GETRANGE key start end](https://www.runoob.com/redis/strings-getrange.html) 返回 key 中字符串值的子字符 |
| ==4==  | [GETSET key value](https://www.runoob.com/redis/strings-getset.html) 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。 |
| 5      | [GETBIT key offset](https://www.runoob.com/redis/strings-getbit.html) 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。 |
| 6      | [MGET key1 [key2..\]](https://www.runoob.com/redis/strings-mget.html) 获取所有(一个或多个)给定 key 的值。 **原子性操作** 所有获取都要成功或者都失败 |
| 7      | [SETBIT key offset value](https://www.runoob.com/redis/strings-setbit.html) 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。 |
| ==8==  | [SETEX key seconds value](https://www.runoob.com/redis/strings-setex.html) 将值 value 关联到 key ，并将 key 的==过期时间==设为 seconds (以秒为单位)。 |
| ==9==  | [SETNX key value](https://www.runoob.com/redis/strings-setnx.html) 只有在 key ==不存在==时设置 key 的值。 |
| ==10== | [SETRANGE key offset value](https://www.runoob.com/redis/strings-setrange.html) 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| 11     | [STRLEN key](https://www.runoob.com/redis/strings-strlen.html) 返回 key 所储存的字符串值的长度。 |
| 12     | [MSET key value [key value ...\]](https://www.runoob.com/redis/strings-mset.html) 同时设置一个或多个 key-value 对。  **原子性操作** 所有获取都要成功或者都失败 |
| 13     | [MSETNX key value [key value ...\]](https://www.runoob.com/redis/strings-msetnx.html) 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。**原子性操作** |
| 14     | [PSETEX key milliseconds value](https://www.runoob.com/redis/strings-psetex.html) 这个命令和 SETEX 命令相似，但它以==毫秒==为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。 |
| 15     | [INCR key](https://www.runoob.com/redis/strings-incr.html) 将 key 中储存的数字值增一。 |
| 16     | [INCRBY key increment](https://www.runoob.com/redis/strings-incrby.html) 将 key 所储存的值加上给定的增量值（increment） 。 |
| 17     | [INCRBYFLOAT key increment](https://www.runoob.com/redis/strings-incrbyfloat.html) 将 key 所储存的值加上给定的浮点增量值（increment） 。 |
| 18     | [DECR key](https://www.runoob.com/redis/strings-decr.html) 将 key 中储存的数字值减一。 |
| 19     | [DECRBY key decrement](https://www.runoob.com/redis/strings-decrby.html) key 所储存的值减去给定的减量值（decrement） 。 |
| 20     | [APPEND key value](https://www.runoob.com/redis/strings-append.html) 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。 |



### List

基本的数据类型。Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

Redis 列表命令

下表列出了列表相关的基本命令：

| 序号   | 命令及描述                                                   |
| :----- | :----------------------------------------------------------- |
| 1      | [BLPOP key1 [key2 \] timeout](https://www.runoob.com/redis/lists-blpop.html) 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 2      | [BRPOP key1 [key2 \] timeout](https://www.runoob.com/redis/lists-brpop.html) 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 3      | [BRPOPLPUSH source destination timeout](https://www.runoob.com/redis/lists-brpoplpush.html) 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 4      | [LINDEX key index](https://www.runoob.com/redis/lists-lindex.html) 通过索引获取列表中的元素 |
| 5      | [LINSERT key BEFORE\|AFTER pivot value](https://www.runoob.com/redis/lists-linsert.html) 在列表的元素前或者后插入元素 |
| 6      | [LLEN key](https://www.runoob.com/redis/lists-llen.html) 获取列表长度 |
| 7      | [LPOP key](https://www.runoob.com/redis/lists-lpop.html) 移出并获取列表的第一个元素 |
| 8      | [LPUSH key value1 [value2\]](https://www.runoob.com/redis/lists-lpush.html) 将一个或多个值插入到列表头部 |
| 9      | [LPUSHX key value](https://www.runoob.com/redis/lists-lpushx.html) 将一个值插入到已存在的列表头部 |
| 10     | [LRANGE key start stop](https://www.runoob.com/redis/lists-lrange.html) 获取列表指定范围内的元素 |
| 11     | [LREM key count value](https://www.runoob.com/redis/lists-lrem.html) 移除列表元素 |
| 12     | [LSET key index value](https://www.runoob.com/redis/lists-lset.html) 通过索引设置列表元素的值 |
| ==13== | [LTRIM key start stop](https://www.runoob.com/redis/lists-ltrim.html) 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被**删除**。类似于Python中的切片 |
| 14     | [RPOP key](https://www.runoob.com/redis/lists-rpop.html) 移除列表的最后一个元素，返回值为移除的元素。 |
| 15     | [RPOPLPUSH source destination](https://www.runoob.com/redis/lists-rpoplpush.html) 移除列表的最后一个元素，并将该元素添加到另一个列表并返回 |
| 16     | [RPUSH key value1 [value2\]](https://www.runoob.com/redis/lists-rpush.html) 在列表中添加一个或多个值 |
| 17     | [RPUSHX key value](https://www.runoob.com/redis/lists-rpushx.html) 为已存在的列表添加值 |

```bash
redis 127.0.0.1:6379> LPUSH runoobkey redis
(integer) 1
redis 127.0.0.1:6379> LPUSH runoobkey mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH runoobkey mysql
(integer) 3
redis 127.0.0.1:6379> LRANGE runoobkey 0 10

1) "mysql"
2) "mongodb"
3) "redis"
```

只用LPUSH或者RPUSH的话，存放类似于stack存储。

LPUSH和RPUSH合起来使用的话，就是一个双端队列。

```bash
127.0.0.1:6379[1]> FLUSHALL
OK
127.0.0.1:6379[1]> LPUSH list1 one
(integer) 1
127.0.0.1:6379[1]> LPUSH list1 two
(integer) 2
127.0.0.1:6379[1]> LPUSH list1 three
(integer) 3
127.0.0.1:6379[1]> RPUSH list1 four
(integer) 4
127.0.0.1:6379[1]> RPUSH list1 five
(integer) 5
127.0.0.1:6379[1]> RPUSH list1 six
(integer) 6
127.0.0.1:6379[1]> LRANGE list1 0 -1
1) "three"
2) "two"
3) "one"
4) "four"
5) "five"
6) "six"
127.0.0.1:6379[1]>

```

![image-20200716124536552](Redis.assets/image-20200716124536552.png)

因为只有LINDEX没有RINDEX，所以只能从左侧开始数。

> 这种结构就可以用来做MQ。





### Set

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

下表列出了 Redis 集合基本命令：

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [SADD key member1 [member2\]](https://www.runoob.com/redis/sets-sadd.html) 向集合添加一个或多个成员    **存** |
| 2    | [SCARD key](https://www.runoob.com/redis/sets-scard.html) 获取集合的成员数     **获取个数** |
| 3    | [SDIFF key1 [key2\]](https://www.runoob.com/redis/sets-sdiff.html) 返回给定所有集合的差集 |
| 4    | [SDIFFSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sdiffstore.html) 返回给定所有集合的差集并存储在 destination 中 |
| 5    | [SINTER key1 [key2\]](https://www.runoob.com/redis/sets-sinter.html) 返回给定所有集合的交集   **存**   case：共同关注 up主 |
| 6    | [SINTERSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sinterstore.html) 返回给定所有集合的交集并存储在 destination 中 |
| 7    | [SISMEMBER key member](https://www.runoob.com/redis/sets-sismember.html) 判断 member 元素是否是集合 key 的成员 |
| 8    | [ SMEMBERS key](https://www.runoob.com/redis/sets-smembers.html) 返回集合中的所有成员   **取** |
| 9    | [SMOVE source destination member](https://www.runoob.com/redis/sets-smove.html) 将 member 元素从 source 集合移动到 destination 集合    **删 + 存** |
| 10   | [SPOP key](https://www.runoob.com/redis/sets-spop.html) 移除并返回集合中的一个**随机**元素   **删** |
| 11   | [SRANDMEMBER key [count\]](https://www.runoob.com/redis/sets-srandmember.html) 返回集合中一个或多个**随机**数       **取** |
| 12   | [SREM key member1 [member2\]](https://www.runoob.com/redis/sets-srem.html) 移除集合中一个或多个成员    **删** |
| 13   | [SUNION key1 [key2\]](https://www.runoob.com/redis/sets-sunion.html) 返回所有给定集合的并集   **取** |
| 14   | [SUNIONSTORE destination key1 [key2\]](https://www.runoob.com/redis/sets-sunionstore.html) 所有给定集合的并集存储在 destination 集合中   **存** |
| 15   | [SSCAN key cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/sets-sscan.html) 迭代集合中的元素 |



> 微博，B站将用户的所有关注放在一个set中，将他的粉丝放到另一个set中。



### Hash

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

下表列出了 redis hash 基本的相关命令：

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [HDEL key field1 [field2\]](https://www.runoob.com/redis/hashes-hdel.html) 删除一个或多个哈希表字段        **删** |
| 2    | [HEXISTS key field](https://www.runoob.com/redis/hashes-hexists.html) 查看哈希表 key 中，指定的字段是否存在。 |
| 3    | [HGET key field](https://www.runoob.com/redis/hashes-hget.html) 获取存储在哈希表中指定字段的值。    **取** |
| 4    | [HGETALL key](https://www.runoob.com/redis/hashes-hgetall.html) 获取在哈希表中指定 key 的所有字段和值      **取**多个 |
| 5    | [HINCRBY key field increment](https://www.runoob.com/redis/hashes-hincrby.html) 为哈希表 key 中的指定字段的**整数值**加上增量 increment 。      **自增**, 设置负数就是减法 |
| 6    | [HINCRBYFLOAT key field increment](https://www.runoob.com/redis/hashes-hincrbyfloat.html) 为哈希表 key 中的指定字段的**浮点数**值加上增量 increment 。 |
| 7    | [HKEYS key](https://www.runoob.com/redis/hashes-hkeys.html) 获取所有哈希表中的字段        **只取key** |
| 8    | [HLEN key](https://www.runoob.com/redis/hashes-hlen.html) 获取哈希表中字段的数量 |
| 9    | [HMGET key field1 [field2\]](https://www.runoob.com/redis/hashes-hmget.html) 获取所有给定字段的值     **取指定的key的value** |
| 10   | ==[HMSET key field1 value1 [field2 value2 \]](https://www.runoob.com/redis/hashes-hmset.html) 同时将多个 field-value (域-值)对设置到哈希表 key 中。== |
| 11   | [HSET key field value](https://www.runoob.com/redis/hashes-hset.html) 将哈希表 key 中的字段 field 的值设为 value 。 |
| 12   | [HSETNX key field value](https://www.runoob.com/redis/hashes-hsetnx.html) 只有在字段 field 不存在时，设置哈希表字段的值。 |
| 13   | [HVALS key](https://www.runoob.com/redis/hashes-hvals.html) 获取哈希表中所有值。     **只取value** |
| 14   | [HSCAN key cursor [MATCH pattern\] [COUNT count]](https://www.runoob.com/redis/hashes-hscan.html) 迭代哈希表中的键值对。 |

```bash
127.0.0.1:6379[1]> HGETALL myhash
1) "name"
2) "kicc"
3) "age"
4) "23"
5) "sex"
6) "male"
7) "nation"
8) "china"
# 获取hash：显示方式 key value key value ...
```

> hash变更的数据user  name  age，尤其是用户信息之类的，经常变动的信息! Hash更适合于对象的存储。
>
> set user:1:name kicc user:1:age 23， 这种存String的方式“存”对象不是那么好。







### Zset

在set的基础上，**增加了一个值**。zset myset score value，增加了一个排序的功能。

 ```bash
#  存储
127.0.0.1:6379[1]> ZADD myset 1 one
(integer) 1
127.0.0.1:6379[1]> zadd myset 2 tow 3 three
(integer) 2
127.0.0.1:6379[1]> ZRANGE myset 0 -1
1) "one"
2) "tow"
3) "three"
127.0.0.1:6379[1]> ZRANGE myset 0 1
1) "one"
2) "tow"
127.0.0.1:6379[1]>

 ```



```bash
# 升序排序
127.0.0.1:6379[1]> zadd salary 5000 xiaoming
(integer) 1
127.0.0.1:6379[1]> zadd salary 2000 xiaohong
(integer) 1
127.0.0.1:6379[1]> zadd salary 500 libai
(integer) 1
127.0.0.1:6379[1]> ZRANGE salary 0 -1
1) "libai"
2) "xiaohong"
3) "xiaoming"
127.0.0.1:6379[1]> ZRANGEBYLEX salary 0 -1
(error) ERR min or max not valid string range item
127.0.0.1:6379[1]> ZRANGEBYLEX salary -inf +inf
(error) ERR min or max not valid string range item

1) "libai"
2) "xiaohong"
3) "xiaoming"
127.0.0.1:6379[1]> ZRANGEBYSCORE salary 0 1000
1) "libai" 
127.0.0.1:6379[1]> ZRANGEBYSCORE salary 0 10000 withscores  # 附带scores
1) "libai"
2) "500"
3) "xiaohong"
4) "2000"
5) "xiaoming"
6) "5000"
127.0.0.1:6379[1]> ZRANGEBYSCORE salary 0 10000 withscores limit 0 1
1) "libai"
2) "500"
127.0.0.1:6379[1]>

# 从大到小排序
127.0.0.1:6379[1]> ZREVRANGE salary 0 -1 withscores
1) "xiaoming"
2) "5000"
3) "xiaohong"
4) "2000"
5) "libai"
6) "500"


######################################################
# 移除zrem

127.0.0.1:6379[1]> ZRANGE salary 0 -1
1) "libai"
2) "xiaohong"
3) "xiaoming"
127.0.0.1:6379[1]> zrem salary xiaoming
(integer) 1
127.0.0.1:6379[1]> ZRANGE salary 0 -1
1) "libai"
2) "xiaohong"
127.0.0.1:6379[1]> ZCARD salary
(integer) 2

```

> 具体的更多的方法根据API看看。

case：

- 普通消息1、重要消息2，进行带权重的判断。
- 排行榜应用的实现。存入一个zset。方便排序比较。

![image-20200716140247704](Redis.assets/image-20200716140247704.png)





## 三种特殊数据类型



### Geospatial 地理空间

朋友的定位，附近的人



![image-20200716140726468](Redis.assets/image-20200716140726468.png)



> 添加地理位置

```bash
# geoadd 添加地理位置
# 一般在Java程序中调用API或者文件全部导入 
127.0.0.1:6379[1]> GEOADD china:city 
116.40 39.90 beijing 
121.47 31.23 shanghai 
106.50 29.53 chongqing 
114.05 22.52 shenzhen 
120.16 30.24 hangzhou
(integer) 5
127.0.0.1:6379[1]>

```



> 获取地理位置 geopos

```bash
127.0.0.1:6379[1]> GEOPOS china:city shenzhen hangzhou
1) 1) "114.04999762773513794"
   2) "22.5200000879503861"
2) 1) "120.1600000262260437"
   2) "30.2400003229490224"

```



> 两个地点之间的距离 geodist

返回两个给定位置之间的距离。

如果两个位置之间的其中一个不存在， 那么命令返回空值。

指定单位的参数 unit 必须是以下单位的其中一个：

- **m** 表示单位为米。 **默认**
- **km** 表示单位为千米。
- **mi** 表示单位为英里。
- **ft** 表示单位为英尺

```bash
127.0.0.1:6379[1]> GEODIST china:city beijing chongqing
"1464070.8051"
127.0.0.1:6379[1]> GEODIST china:city beijing chongqing km
"1464.0708"

```



> georadius

我附近的人。通过半径来查询

以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素

```bash
# 以 120 30 为中心 找 china:city 中 500km内的点
127.0.0.1:6379[1]> GEORADIUS china:city 120 30 500 km
1) "hangzhou"
2) "shanghai"
127.0.0.1:6379[1]>

```



> GEORADIUSBYMEMBER

这个命令和 [GEORADIUS](http://www.redis.cn/commands/georadius.html) 命令一样， 都可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 [GEORADIUS](http://www.redis.cn/commands/georadius.html) 那样， 使用输入的经度和纬度来决定中心点



> GEO底层的实现原理：就是ZSET

```bash
# 查看所有
127.0.0.1:6379[1]> ZRANGE china:city 0 -1
1) "chongqing"
2) "shenzhen"
3) "hangzhou"
4) "shanghai"
5) "beijing"
# 移除指定元素
127.0.0.1:6379[1]> zrem china:city shenzhen
(integer) 1

```



### Hyperloglog：用来计数

> 什么是基数

A {1, 3, 5, 7, 9, 7}

B {1, 3, 5, 7, 8}

基数（不重复的元素） = 9，可接受误差。



> 简介

Redis 2.8.9 更新了hyperloglog的数据结构！它是用来做基数统计的算法！

HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

下表列出了 redis HyperLogLog 的基本命令：PF开头

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PFADD key element [element ...\]](https://www.runoob.com/redis/hyperloglog-pfadd.html) 添加指定元素到 HyperLogLog 中。 |
| 2    | [PFCOUNT key [key ...\]](https://www.runoob.com/redis/hyperloglog-pfcount.html) 返回给定 HyperLogLog 的基数估算值。 |
| 3    | [PFMERGE destkey sourcekey [sourcekey ...\]](https://www.runoob.com/redis/hyperloglog-pfmerge.html) 将多个 HyperLogLog 合并为一个 HyperLogLog |

```bash
127.0.0.1:6379[1]> PFADD mykey1 a b c d e f g h i j  # 创建第一组元素
(integer) 1
127.0.0.1:6379[1]> PFCOUNT mykey1	# 统计基数
(integer) 10
127.0.0.1:6379[1]> PFADD mykey2 i k a f g ga w e r h sd x c z a q	# 创建第一组元素
(integer) 1
127.0.0.1:6379[1]> PFCOUNT mykey2	# 统计基数
(integer) 15
127.0.0.1:6379[1]> PFMERGE mykey3 mykey1 mykey2	# 合并两个hyperloglog
OK
127.0.0.1:6379[1]> PFCOUNT mykey3	# 不重复的元素
(integer) 18

```

应用场景：

- 统计计数，比如用户的访问数



如果允许误差错误，那么就可以使用Hyperloglog，大数据下更有效。

如果不允许一点错误，就使用set或者自己的数据类型。





### Bitmap

> 两个状态码的场景就很适用。

case：

- 统计用户信息，活跃、不活跃。

- 登录、未登录。
- 365天打卡。365个0，打卡就是1，不是打卡就0。

![image-20200716153019815](Redis.assets/image-20200716153019815.png)

假设统计一周七天的打卡情况，没打卡就是0， 打卡了就是1.

```bash
setbit key offset value
```

上图表示的意思就是0-5天都打卡了，但是第7天没有打卡。

现在我需要统计打卡的总天数，我只要获取多少个1就可以了。！



**查看某一天是否打卡：**

```bash
127.0.0.1:6379[1]> GETBIT daka 4
(integer) 1
127.0.0.1:6379[1]> GETBIT daka 6
(integer) 0
127.0.0.1:6379[1]> GETBIT daka 5
(integer) 1

```



**统计打卡的天数：**

```bash
# 统计这周打卡记录
127.0.0.1:6379[1]> BITCOUNT daka
(integer) 6

```



## 事务

Redis事务的本质：一组命令的集合！一组事务中的所有命令都会被序列化，在事务执行过程中，会按顺序执行！

一次性、顺序性、排他性。执行一些命令。

```
------ 队列   set   set   set  执行 -------
```

==Redis事务没有隔离级别的概念！==

所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会被执行。  Exec

==Redis单挑命令是保存原子性的，但是事务不保证原子性！==

Redis事务：

- 开启事务（multi）
- 命令入队（）
- 执行事务（Exec）

> 正常执行一组事务

```bash
# 执行一组事务
127.0.0.1:6379[1]> multi
OK
127.0.0.1:6379[1]> set k1 v1
QUEUED
127.0.0.1:6379[1]> set k2 v2
QUEUED
127.0.0.1:6379[1]> get k1
QUEUED
127.0.0.1:6379[1]> set k3 v3
QUEUED
127.0.0.1:6379[1]> exec
1) OK
2) OK
3) "v1"
4) OK

```

> 放弃一组事务

```bash

127.0.0.1:6379[1]> multi
OK
127.0.0.1:6379[1]> set k1 v1
QUEUED
127.0.0.1:6379[1]> set k2 v2
QUEUED
127.0.0.1:6379[1]> get k1
QUEUED
127.0.0.1:6379[1]> set k4 v4
QUEUED
127.0.0.1:6379[1]> discard
OK
127.0.0.1:6379[1]> get k4
(nil)
```



> 编译型异常（代码有问题、命令有错），事务中所有的命令都不会被执行



> 运行时异常（比如1/0），如果事务队列中存在语法性错误，那么执行命令的时候，其他命令是可以正常执行的。错误的那句不执行。





> 监控！

乐观锁：

- 很乐观。认为什么时候都不会出现问题。就不会加锁。更新数据的时候判断一下，在过程中是否有人修改过数据。**（每次比较一个version字段）**
- 获取version
- 更新的时候比较version是否有无改动



悲观锁：

- 很悲观！什么时候都会出现问题。必须加锁。（加锁肯定影响性能）

> Redis的监控  watch 命令 是一种乐观锁

**一、正常的执行**

```bash
127.0.0.1:6379[1]> set money 100
OK
127.0.0.1:6379[1]> set out 0
OK
127.0.0.1:6379[1]> watch money   # 监视money对象
OK
127.0.0.1:6379[1]> multi 	# 事务正常结束，数据期间没有发生变动
OK
127.0.0.1:6379[1]> DECRBY money 20
QUEUED
127.0.0.1:6379[1]> INCRBY out 20
QUEUED
127.0.0.1:6379[1]> exec
1) (integer) 80
2) (integer) 20

```



**二、测试多线程修改值**

```bash
127.0.0.1:6379[1]> watch money   # 监视money对象
OK
127.0.0.1:6379[1]> multi 	# 事务还没有执行exec，但是有另一个线程先改变了money的值，事务就会执行失败
OK
127.0.0.1:6379[1]> DECRBY money 20
QUEUED
127.0.0.1:6379[1]> INCRBY out 20
QUEUED
127.0.0.1:6379[1]> exec  # 通过监视money对象，发现exec的时候跟进入事务的时候money已经被改变了！ 所以不能执行成功
(nil)
127.0.0.1:6379[1]> unwatch   # 放弃之前的监视
OK
```

执行失败怎么办？

1. unwatch 放弃之间对对象的监视
2. 再次watch监视
3. 重新执行之前的事务（再过一遍）





## Jedis

使用Jedis来操作Redis

> 什么是Jedis 是Redis官方推荐的Java连接开发工具！使用Java操作Redis中间件！如果你要使用Java操作Redis，那么一定要对Jedis十分的熟悉！

```xml
依赖导入
<dependencies>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.70</version>
    </dependency>
</dependencies>
```



测试连接

```java
public class TestPing {

    public static void main(String[] args) {
        // 1、 new jedis对象
        Jedis jedis = new Jedis("192.168.1.114", 6379);
        System.out.println(jedis);
        String name = jedis.get("name");
        System.out.println(name);

        System.out.println(jedis.ping());

    }
}
```

> Jedis的所有方法和redis-cli的命令都是一一对应的。



**Jedis操作事务**

```java
public class TestPing {

    public static void main(String[] args) {
        // 1、 new jedis对象
        Jedis jedis = new Jedis("192.168.1.114", 6379);
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name", "kicc");
        jsonObject.put("age", 23);


        // 开启事务
        Transaction multi = jedis.multi();
		// 如果要监控
        jedis.watch()
        try {
			
            multi.set("user1", jsonObject.toJSONString());
            // 执行
            multi.exec();
        } catch (Exception e) {
            // 放弃事务
            multi.discard();
            e.printStackTrace();
        } finally {
            jedis.close();
        }
        
    }
}
```



## SpringBoot整合

spring-security、spring-data和springboot是同一层级的框架。

说明：在SpringBoot2.x之后，原来使用的jedis被替换为lettuce？

jedis：采用的直连，多个线程操作的话，是不安全的。如果想要避免不安全的，使用jedis pool连接池！ 更像BIO

lettuce：采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况！可以减少线程数量。更新NIo



> 整合测试一下

老规矩，先看看源码

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

   @Bean
   @ConditionalOnMissingBean(name = "redisTemplate") // 如果我们自定义一个redisTemplate，就可以把这个覆盖
   // 默认的redisTempalte没有过多的设置，不实现序列化
   // 两个泛型都是Object，我们使用需要强转为<String, Object>
   public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
         throws UnknownHostException {
      RedisTemplate<Object, Object> template = new RedisTemplate<>();
      template.setConnectionFactory(redisConnectionFactory);
      return template;
   }

   @Bean
   @ConditionalOnMissingBean  // 因为最常用的是String类型的redis，因此额外多一个StringRedisTemplate
   public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
         throws UnknownHostException {
      StringRedisTemplate template = new StringRedisTemplate();
      template.setConnectionFactory(redisConnectionFactory);
      return template;
   }

}
```



看看用法：

![image-20200716204502794](Redis.assets/image-20200716204502794.png)

.opsForXxxx用于操作不同的数据类型。ForValue是操作String类型。



- 获取redis的连接

    ```java
    RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
    // 用connection操作flushdb flushAll操作！
    connection.flushAll();
    ```

- 来个get set

    ```java
    @Test
    void contextLoads() {
        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        connection.flushAll();
        
        redisTemplate.opsForValue().set("name", "kicc");
        redisTemplate.opsForValue().get("name");
    }
    ```





![image-20200716210603214](Redis.assets/image-20200716210603214.png)

![image-20200716210648760](Redis.assets/image-20200716210648760.png)

> 因为默认的ReidsTemplate配置不是很多，所以最好是自己配置一下。



真实开发中，都传递Json对象。

```java
@Test
void test() {

    User kicc = new User("Kicc", "123456...");

    String s = JSON.toJSONString(kicc);
    // 存入一个user序列化为Json的String
    redisTemplate.opsForValue().set("user", s);
    
    Object user = redisTemplate.opsForValue().get("user");
    System.out.println(user);

}
```

查看Redis-cli

![image-20200716212748096](Redis.assets/image-20200716212748096.png)

**显示是用问题的。**

解决办法：

- 将redisTemplate的key的序列化方式改为StringRedisSerialize

    ![image-20200716214052569](Redis.assets/image-20200716214052569.png)

- 直接使用StringRedisTemplate 

- 我们需要编写一个自己的redis配置。

    ```java
    @Configuration
    public class RedisConfig {
    
        /**
         *
         * @param redisConnectionFactory
         * @return
         * @throws UnknownHostException
         */
        @Bean
        public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
                throws UnknownHostException {
            RedisTemplate<String, Object> template = new RedisTemplate<>();
            template.setConnectionFactory(redisConnectionFactory);
    
            // FastJson序列化配置，不需要ObjectMapper
            FastJsonRedisSerializer fastJsonRedisSerializer = new FastJsonRedisSerializer(Object.class);
    
    
            ObjectMapper objectMapper = new ObjectMapper();
            objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
            //objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL); 这个方法在jackson Since 2.10版本中移除掉了
            objectMapper.activateDefaultTyping(BasicPolymorphicTypeValidator.builder().build(),ObjectMapper.DefaultTyping.NON_FINAL);
    
            Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
            jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
    
    		// 给key设置String的序列化方式
            template.setKeySerializer(new StringRedisSerializer());
            // 给value设置String的序列化方式， 目前没看出了fastJson的序列化和JackSon的区别
            template.setValueSerializer(jackson2JsonRedisSerializer);
            // 给hashkey设置String的序列化方式
            template.setHashKeySerializer(new StringRedisSerializer());
            // 给hashValue设置String的序列化方式
            template.setHashValueSerializer(jackson2JsonRedisSerializer);
    		
            // 加载配置
            template.afterPropertiesSet();
    
    
            return template;
        }
    }
    ```



**提取最常用的操作编写RedisUtils工具类，过一遍很适合把RedisTemplate快速掌握！**

![image-20200716221711833](Redis.assets/image-20200716221711833.png)









## Redis.conf详解

配置时，都靠配置文件来启动！

> 内存大小设置，大小写不敏感

![image-20200717162013922](Redis.assets/image-20200717162013922.png)

> 包含多个配置文件

![image-20200717162056703](Redis.assets/image-20200717162056703.png)



> 网络

一、绑定的IP

![image-20200717162237852](Redis.assets/image-20200717162237852.png)



二、



> 通用

```bash
daemonize yes # 后台守护进程方式
pidfile /var/run/redis_6379.pid  # 如果以后台方式运行，我们就需要指定一个pid进程文件

# 日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)  生产环境使用
# warning (only very important / critical messages are logged)	
loglevel notice

# 文件名 ”“表示标准输出
logfile ""


```



> 快照

持久化操作。在规定的时间内，执行了多少次操作，则会持久化到文件。 .rdb 和 .aof文件。

redis 是 内存数据库，如果没有持久化，数据断电即失。

```bash

# 
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1   # 900秒 超过一个key修改，进行持久化
save 300 10  # 30秒内 超过10个key修改，进行持久化
save 60 10000 # 高并发下的持久化策略

# 
stop-writes-on-bgsave-error yes

# 是否压缩rdb文件，需要消耗cpu
rdbcompression yes
# 检查rdb文件的校验
rdbchecksum yes

# rdb的文件和路径
dbfilename dump.rdb
dir ./

```



> Replication 主从复制







> Security 安全

```bash
# 密码的设置
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> config get requiredpass
(empty list or set)
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass 123456
OK
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> ping
PONG

```



> 客户端限制

```bash

# maxclients 10000

# 如果内存满了，怎么处理
# maxmemory-policy noeviction
# 6个策略
noeviction:默认策略，不淘汰，如果内存已满，添加数据是报错。
allkeys-lru:在所有键中，选取最近最少使用的数据抛弃。
volatile-lru:在设置了过期时间的所有键中，选取最近最少使用的数据抛弃。
allkeys-random: 在所有键中，随机抛弃。
volatile-random: 在设置了过期时间的所有键，随机抛弃。
volatile-ttl:在设置了过期时间的所有键，抛弃存活时间最短的数据。
```



> APPEND ONLY MODE aof模式

```bash
# 默认使用rdb持久化，所以不开启aof。一般rdb够用了。
appendonly no

# 默认文件名
appendfilename "appendonly.aof"

# 同步策略
# appendfsync always
appendfsync everysec
# appendfsync no


```









## Redis持久化

内存数据化必须要有的功能。

#### RDB

![img](https://user-gold-cdn.xitu.io/2019/6/26/16b914b543343855?imageslim)



在指定时间间隔内将内存中的数据集快照写入磁盘，恢复时将快照文件直接读入内存。

> 如何操作？

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化文件。整个过程中，主进程不进行IO操作。保证的性能。如果需要进行大规模的数据恢复，且对于数据恢复的完整性==不是非常敏感==，那么RDB的方式比AOF更加高效。RDB的缺点就是最后一次持久化之后的数据可能丢失。

RDB保存的文件是 dump.rdb

> 触发规则

1、save的规则满足的条件下，会自动触发rdb规则

2、执行flushall命令时

3、退出redis



> 恢复rdb文件

只需要将rdb文件放在我们redis启动目录下就可以，redis启动的时候会自动检查dump.rdb。不需要手动导入。

```bash

127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin"
127.0.0.1:6379>

```

优点：

- 适合大规模的数据恢复
- 对数据的完整性要求不高（万一最后一次还没来得及持久化）

缺点：

- 需要一定的时间间隔修改； 万一宕机，最后一次数据的修改就没了
- fork时，会占用一定内存空间



#### AOF （Append Only File)

将我们所有的命令都记录下来，history，恢复的时候重新把命令执行一遍。

> 使用 AOF 做持久化，每一个写命令都通过write函数追加到 appendonly.aof 中,

![img](https://user-gold-cdn.xitu.io/2019/6/26/16b916ccf4224ec3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来**（读操作不记录）**，只许追加文件但不可以改写文件，redis驱动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将指令从前到后执行一次以完成数据的恢复工作

==Aof保存的是 appendonly.aof文件==。大规模数据情况下，恢复起来比较慢。

redis.conf默认不开启aof，需要手动开启。



来看一下appendonly.aof长什么样子

![image-20200717174844874](Redis.assets/image-20200717174844874.png)

```bash
# 查看appendonly.aof文件
*2
$6
SELECT
$1
0
*3
$3
set
$2
k1
$2
v1
*3
$3
set
$2
k2
$2
v2
*3
$3
set
$2
k3
$2
v3

```

我们插入了三个key-value，查看.aof文件后发现，保存的就是所有的操作记录。	

假设现在我们人为修改了aof文件，文件毫无疑问地遭到了破坏！那么我们重启redis-server之后是无法直接连接上的！（因为有错误！）

此时，我们需要使用 redis-check-aof 来帮助恢复。

![image-20200717175101536](Redis.assets/image-20200717175101536.png)

```bash
# 修复
redis-check-aof --fix appendonly.aof
```





> 优点和缺点

优点：

- 每一次修改都同步 （默认每秒修改）。文件的完整性更好

缺点：

- 性能不高，aof修复的速度远远不如rdb。
- Aof的运行效率也不行。（写操作）





#### 两个的直观比较

![img](https://user-gold-cdn.xitu.io/2019/6/26/16b918cd860b0ffd?imageslim)



**区别：**

- RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

- AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。





#### 使用建议

当同时开启RDB和AOF时，redis重启后会优先载入AOF文件来恢复数据。因为通常情况下，AOF文件保存的数据要比RDB更加完整。

> 性能建议

- 因为RDV文件只用作后备用途，建议只在Slave上持久化RDB文件。而且15分钟一次备份就够。只保留save 900 1这条规则。
- 如果Enable AOF，好处是最恶劣情况下也只会丢失不超过两秒的数据，启动脚本较简单只load自己的aof文件就可以。代价是带来了持续的IO，**二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。**只要硬盘许可，应该尽量减少AOF rewrite的频率。AOF重写的基础大小默认是64M，太小了！可以设置为5G，默认超过原大小100%大小重写可以改到适当的百分数。
- 如果不Enable AOF，仅仅靠Master-Slave Replication 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时候带来的系统波动。代价是如果Master和Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master\Slave中的RDB文件，载入较新的哪个，微博就是这种架构。





## Redis发布订阅



## Redis主从复制

高可用，哨兵模式

## Redis缓存穿透和雪崩





