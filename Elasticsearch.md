# Elasticsearch笔记

[开源搜索：Elasticsearch、ELK Stack 和 Kibana 的开发者 | Elastic](https://www.elastic.co/cn/)

# Elasticsearch简介

全文搜索属于最常见的需求，开源的Elasticsearch是目前全文搜索引擎的首选。它可以快速地存储、搜索和分析海量数据。维基百科、Stack Overflow、Github都采用它。

Elastic的底层是开源库Lucene。但是，你没法直接用Lucene，必须自己写代码去调用它的接口。Elastic是Lucene的封装，提供了REST API的操作接口，开箱即用。

# 基本概念

1. Index（索引）

   当做动词，相当于MySQL中的insert

   当做名次，相当于MySQL中的Database

2. Type（类型）

   在Index（索引）中，可以定义一个或多个类型。类似于MySQL中的Table表。每一种类型的数据放在一起。

3. Document（文档）

   保存在某个索引（Index）下，某种类型（Type）的一个数据（Document），文档是JSON格式的，Document就像是MySQL中的某个Table的内容。

4. 倒排索引

   Elasticsearch 使用的是一种名为_倒排索引_的数据结构，这一结构的设计可以允许十分快速地进行全文本搜索。倒排索引会列出在所有文档中出现的每个特有词汇，并且可以找到包含每个词汇的全部文档。

   在索引过程中，Elasticsearch 会存储文档并构建倒排索引，这样用户便可以近实时地对文档数据进行搜索。索引过程是在索引 API 中启动的，通过此 API 您既可向特定索引中添加 JSON 文档，也可更改特定索引中的 JSON 文档。

# Docker安装

1. 下载镜像文件

   ```bash
   docker pull elasticsearch:7.4.2
   // 可视化检索工具
   docker pull kibana:7.4.2 
   ```

2. Elasticsearch

   ```bash
   mkdir -p /mydata/elasticsearch/config
   mkdir -p /mydata/elasticsearch/data
   echo "http.host: 0.0.0.0">>/mydata/elasticsearch/config/elasticsearch.yml 
   
   
   docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
   -e "discovery.type=single-node" \
   -e ES_JAVA_OPTS="-Xms64m -Xmx128m" \
   -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
   -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
   -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
   -d elasticsearch:7.4.2 
   ```

   特别注意：

   -e ES_JAVA_OPTS="Xms64m -Xmx128m" 测试环境下，设置ES的初始内存和最大内存，否则导致过大启动不了ES

   修改data、config、plugins目录的权限

   ```bash
   chmod -R 777 /mydata/elasticsearch/ 
   ```

3. Kibana

   ```bash
   docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 \
   -d kibana:7.4.2 
   ```

# 初步检索

1. _cat

   GET /_cat/nodes→查看所有节点

   GET /_cat/health→查看es健康状况

   GET /_cat/master→查看主节点

   GET /_cat/indices→查看所有索引

2. 索引一个文档（保存）

   保存一个数据，保存在哪个索引的哪个类型下，指定用哪个唯一标识

   PUT customer/external/1→在customer索引下的external类型下保存1好数据为

   ```json
   http://192.168.56.10:9200/customer/external/1
   {
     "name":"zhangsan"
   } 
   ```

   发送多次是更新操作

   ```json
   {
       "_index": "customer",
       "_type": "external",
       "_id": "1",
       "_version": 1,
       "result": "created",
       "_shards": {
           "total": 2,
           "successful": 1,
           "failed": 0
       },
       "_seq_no": 0,
       "_primary_term": 1
   }
   ```

   

   post→http://192.168.56.10:9200/customer/external/ 不指定id则一直created，并自动生成唯一id，如果指定了id则是更新

3. 查询文档

   GET customer/external/1

   http://192.168.56.10:9200/customer/external/1

   ```json
   {
       "_index": "customer", // 在哪个索引
       "_type": "external", // 在哪个类型
       "_id": "1", // 记录id
       "_version": 2, // 版本号
       "_seq_no": 1, // 并发控制字段，每次更新就会+1，用来做乐观锁
       "_primary_term": 1, // 同上，住分片重新分配，如重启，就会变化
       "found": true, // 是否找到
       "_source": { // 真正的内容
           "name": "zhangsan"
       }
   }
   ```

   

   更新携带 →?if_seq_no=1&if_primary_term=1

4. 更新文档

   ```json
   POST customer/external/1/_update
   {
     "doc":{
       "name":"lisi"
     }
   }
   // 会对比元数据是否有改变如果没有则不做任何操作
   ```

   ```json
   POST customer/external/1
   {
      "name":"lisi"
   }
   // 不会对比元数据是否有改变直接更新
   ```

   ```json
   PUT customer/external/1
   {
      "name":"lisi"
   }
   // 不会对比元数据是否有改变直接更新 
   ```

5. 删除文档&索引

   DELETE customer/external1

   DELETE customer

   不支持删除类型

6. bulk批量API

   ```json
   POST customer/external/_bulk
   {"index":{"_id":"1"}}
   {"name":"zhangsan"}
   {"index":{"_id":"2"}}
   {"name":"lisi"} 
   {"create":{"_id":"1"}}
   {"name":"lisi"}
   // 语法格式
   {"action":{metadata}} \n
   {request body}\n
   {"action":{metadata}} \n
   {request body}\n
   ```

   ```json
   {
     "took" : 350,
     "errors" : false,
     "items" : [
       {
         "index" : {
           "_index" : "customer",
           "_type" : "external",
           "_id" : "1",
           "_version" : 1,
           "result" : "created",
           "_shards" : {
             "total" : 2,
             "successful" : 1,
             "failed" : 0
           },
           "_seq_no" : 0,
           "_primary_term" : 1,
           "status" : 201
         }
       },
       {
         "index" : {
           "_index" : "customer",
           "_type" : "external",
           "_id" : "2",
           "_version" : 1,
           "result" : "created",
           "_shards" : {
             "total" : 2,
             "successful" : 1,
             "failed" : 0
           },
           "_seq_no" : 1,
           "_primary_term" : 1,
           "status" : 201
         }
       }
     ]
   }
   ```

   注意：create 不会覆盖文档，index会覆盖文档进行更新

7. 导入测试数据

   [accounts.zip - 蓝奏云](https://wws.lanzous.com/i0ptsjzxi2h)

   ```json
   POST /bank/account/_bulk
   {
     ....
   } 
   ```

   