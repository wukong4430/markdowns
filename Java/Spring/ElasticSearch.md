# What is ElasticSearch

我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前**全文搜索引擎**的 首选。

Elasticsearch是一个**开源**的**高可扩展**的**分布式**搜索服务，提供Restful API，底层基于Lucene，采用 多shard(分片)的方式保证数据安全，并且提供自动resharding的功能，github 等大型的站点也是采用了ElasticSearch作为其搜索服务。



# ES与Solr的区别



### ElasticSearch简介

开源的、高可扩展、分布式搜索框架。

适用于大数据。



### Solr简介

Java开发、基于Lucence

用POST方法向Solr服务器发送一个描述Field及其内容的XML文档，Solr根据xml文档添加、删除、更新索引。

用GET请求，对Solr返回Xml、Json等格式的查询结果进行解析，组织页面布局。Solr不提供构建UI的功能，Solr提供了一个管理界面，通过管理界面可以查询Solr的配置和运行情况。





## 什么是ELK

ElasticSearch + Logstash + Kibana

- Logstash是ELK中的中央数据流引擎，用于从不同目标（文件/数据存储/MQ）收集不同的格式数据，经过过滤后支持输出到不同目的地（文件/MQ/Redis/Es/Kafka等）。
- Kibana可以将ES的数据通过友好的界面展示出来，提供数据实时分析功能







# ElasticSearch 安装

1. #### 第一步 拉取镜像

```bash
docker pull elasticsearch:7.8.0
```

2. #### 第二步 运行Elasticsearch

```bash
# -p 代表映射端口，这里我们将es的9200和9300这两个需要用到的端口映射成相同的外部端口号，discovery.type这边使用的是开发环境的配置，single-node代表是单节点非集群，--name 给容器取一个别名。
sudo docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```

3. #### 第三步 检查是否运行成功

访问 `http://localhost:9200` ，默认情况下无法外网访问。可以另开`SSH`窗口，运行`curl http://localhost:9200`

4. #### 第四步 守护进程运行 (后台运行)

```bash
sudo docker run -itd -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```





# Kibana 安装

一、拉取镜像

```txt
docker pull docker.elastic.co/kibana/kibana:7.8.0
```



二、启动

```bash
docker run --link YOUR_ELASTICSEARCH_CONTAINER_NAME_OR_ID:elasticsearch -p 5601:5601 {docker-repo}:{version}

docker run --link es01:elasticsearch -p 5601:5601 kibana:7.8.0
```



三、测试访问

IP:5601



四、控制台代码编写

![image-20200811132641283](ElasticSearch.assets/image-20200811132641283.png)



# ES-head 安装

> 可视化界面

**一、安装**

```bash
docker run --name es-head -p 9100:9100 mobz/elasticsearch-head:5
```



**二、支持跨域**

在容器 running下，进入容器

```bash
docker exec -it es01 bash
```



编辑 `config/elasticsearch.yml`

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```



> 问题：数据浏览406

字符编码格式错误！



解决办法：docker

- ```bash
    docker exec -it ... /bin/bash # 进入容器
    ```

- ```
    cd _site
    ```

- ```bash
    vim vendor.js
    ```

- 将 6886行 contentType: "application/x-www-form-urlencoded" 修改为 contentType: "application/json;charset=UTF-8"
    然后再将 7574行 var inspectData = s.contentType === "application/x-www-form-urlencoded" && 修改为 var inspectData = s.contentType === "application/json;charset=UTF-8" &&

- 刷新浏览器



### ES-head的作用

我们一般只用ES-head来查看数据，不做查询。因为查询是用Json的，ES-head中编写Json不方便。

查询就用Kibana，查询更加方便。





# ES 核心

知道了如何使用 ES 之后， 就来学习一下ES的数据结构、实现原理吧

==集群、节点、索引、类型、文档、分片、映射== 都是什么？

> elasticsearch是面向文档的。以下给出了关系型数据库和 ES的客观对比！



| RDB                 | ES                 |
| ------------------- | ------------------ |
| 数据库 （database） | 索引（indices）    |
| 表（tables）        | 类型 （types）     |
| 行（rows）          | 文档 （documents） |
| 字段（colums）      | fields             |



1、索引

2、字段类型

3、文档

4、Lucence索引（倒排索引、分片）





# IK分词器

用于中文分词



### 手动压缩包 安装

一、安装

https://github.com/medcl/elasticsearch-analysis-ik



二、解压到ES的plugin下



### docker install IK

```shell
# 进入容器es
docker exec -it es /bin/bash
# 使用bin目录下的elasticsearch-plugin install安装ik插件
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.0/elasticsearch-analysis-ik-7.8.0.zip
# 再重启下容器
docker restart es
```



上面的docker直接安装速度会很慢，可以将zip文件拷贝到docker容器中

```bash
# ik 是 解压后的文件夹
docker cp ik es01:/usr/share/elasticsearch/plugins/
```







### 验证分词器效果

```shell
# 增加一个叫test001的索引
curl -X PUT http://localhost:9200/test001
# 成功返回 {"acknowledged":true,"shards_acknowledged":true,"index":"test001"}

# ik_smart分词
curl -X POST \
'http://127.0.0.1:9200/test001/_analyze?pretty=true' \
-H 'Content-Type: application/json' \
-d '{"text":"我们是软件工程师","tokenizer":"ik_smart"}'

# ik_max_word分词
curl -X POST \
'http://127.0.0.1:9200/test001/_analyze?pretty=true' \
-H 'Content-Type: application/json' \
-d '{"text":"我们是软件工程师","tokenizer":"ik_max_word"}'
```

![img](ElasticSearch.assets/15296782-a5d10c6e19a0054b.png)



![img](ElasticSearch.assets/15296782-595081e4e7295cc1.png)



### 在Kibana中使用

- **ik_smart**

![image-20200811135837904](ElasticSearch.assets/image-20200811135837904.png)

- **ik_max_word**

![image-20200811135925558](ElasticSearch.assets/image-20200811135925558.png)



### 自定义关键词

观察  `狂神说`

因为没有一个字典会记录 `狂神说`，所以我们需要手动的添加一个包含 `狂神说`的字典，并配置的IK分词器中。

![image-20200811140320887](ElasticSearch.assets/image-20200811140320887.png)



现在就可以直接识别 `狂神说`

![image-20200811140351095](ElasticSearch.assets/image-20200811140351095.png)



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

