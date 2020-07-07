# What is ElasticSearch

我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前全文搜索引擎的 首选。

Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用 多shard(分片)的方式保证数据安全，并且提供自动resharding的功能，github 等大型的站点也是采用了ElasticSearch作为其搜索服务。





# ElasticSearch 安装

1. #### 第一步 拉取镜像

```bash
docker pull elasticsearch:7.8.0
```

2. #### 第二步 运行Elasticsearch

```bash
sudo docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```

3. #### 第三步 检查是否运行成功

访问 `http://localhost:9200` ，默认情况下无法外网访问。可以另开`SSH`窗口，运行`curl http://localhost:9200`

4. #### 第四步 守护进程运行 (后台运行)

```bash
sudo docker run -itd -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.8.0
```





# Kibana 安装







# ES-head 安装

**一、安装**

```bash
docker run -p 9100:9100 mobz/elasticsearch-head:5
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

