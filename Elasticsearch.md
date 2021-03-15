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
   -e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
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




# 进阶检索

1. SearchAPI

   [API]( https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-search.html)

   1. ES支持两种基本方式检索

      - 一个是通过使用REST request URI 发送搜索参数（URI+检索参数）

        ```
        请求参数方式检索
        GET bank/_search?q=*&sort=account_number:asc
        
        检索bank下所有信息，包括type和docs
        GET bank/_search
        ```

        - took – 花费多少ms搜索
        - timed_out – 是否超时
        - __shards – 多少分片被搜索了，以及多少成功/失败的搜索分片_
        - max_score –文档相关性最高得分
        - hits.total.value - 多少匹配文档被找到
        - hits.sort - 结果的排序key，没有的话按照score排序
        - hits._score - 相关得分 (not applicable when using match_all)

        ```
        q=* 查询所有
        sort 排序字段
        asc升序
        ```

        ```
        GET bank/_search?q=*&sort=account_number:asc
        
        检索了1000条数据，但是根据相关性算法，只返回10条
        ```

        

      - 另一个是通过使用REST request body 来发送他们（URI+请求体）

        ```
        GET /bank/_search
        {
          "query": { "match_all": {} },
          "sort": [
            { "account_number": "asc" },
            { "balance":"desc"}
          ]
        }
        ```

        > postman中get不能携带请求体，我们变为post也是一样的，我们post一个jsob风格的查询请求体到_search
        >
        > 需要了解，一旦搜索的结果被返回，es就完成了这次请求，不能切不会维护任何服务端的资源或者结果的cursor游标

   2. Query DSL 

      1. 基本语法格式

         Elasticsearch提供了一个可以执行查询的Json风格的DSL(domain-specific language领域特定语言)

         - 一个查询语句的典型结构

           ```
           QUERY_NAME:{
              ARGUMENT:VALUE,
              ARGUMENT:VALUE,
               ...
           }
               
           如果针对于某个字段，那么它的结构如下：
           {
             QUERY_NAME:{
                FIELD_NAME:{
                  ARGUMENT:VALUE,
                  ARGUMENT:VALUE,...
                 }   
              }
           }
           ```

           ```
           示例
           GET bank/_search
           {
             "query": {
               "match_all": {}
             },
             "from": 0,
             "size": 5,
             "_source":["balance"],
             "sort": [
               {
                 "account_number": {
                   "order": "desc"
                 }
               }
             ]
           }
           _source为要返回的字段
           ```

           

         - query定义如何查询

           - match_all查询类型【代表查询所有的所有】，es中可以在query中组合非常多的查询类型完成复杂查询；
           - 除了query参数之外，我们可也传递其他的参数以改变查询结果，如sort，size；
           - from+size限定，完成分页功能；
           - sort排序，多字段排序，会在前序字段相等时后续字段内部排序，否则以前序为准；

      2. 返回部分字段

         ```json
         GET bank/_search
         {
           "query": {
             "match_all": {}
           },
           "from": 0,
           "size": 5,
           "sort": [
             {
               "account_number": {
                 "order": "desc"
               }
             }
           ],
           "_source": ["balance","firstname"]
           
         }
         ```

         ```json
         查询结果
         {
           "took" : 18,
           "timed_out" : false,
           "_shards" : {
             "total" : 1,
             "successful" : 1,
             "skipped" : 0,
             "failed" : 0
           },
           "hits" : {
             "total" : {
               "value" : 1000,
               "relation" : "eq"
             },
             "max_score" : null,
             "hits" : [
               {
                 "_index" : "bank",
                 "_type" : "account",
                 "_id" : "999",
                 "_score" : null,
                 "_source" : {
                   "firstname" : "Dorothy",
                   "balance" : 6087
                 },
                 "sort" : [
                   999
                 ]
               },
               省略。。。
         
         ```

      3. match匹配查询

         - 基本类型（非字符串），精确控制

           ```json
           GET bank/_search
           {
             "query": {
               "match": {
                 "account_number": "20"
               }
             }
           }
           ```

           match返回account_number=20的数据。

         - 字符串，全文检索

           ```json
           GET bank/_search
           {
             "query": {
               "match": {
                 "address": "kings"
               }
             }
           }
           ```

           全文检索，最终会按照评分进行排序，会对检索条件进行分词匹配。

      4. match_phrase短语匹配

         - 将需要匹配的值当成一整个单词（不分词）进行检索

           ```json
           GET bank/_search
           {
             "query": {
               "match_phrase": {
                 "address": "mill road"
               }
             }
           }
           ```

           > 前面的是包含mill或road就查出来，我们现在要都包含才查出

           查出address中包含mill road的所有记录，并给出相关性得分

      5. multi_math【多字段匹配】

         - **state或者address中包含mill**，并且在查询过程中，会对于查询条件进行分词

           ```json
           GET bank/_search
           {
             "query": {
               "multi_match": {
                 "query": "mill",
                 "fields": [
                   "state",
                   "address"
                 ]
               }
             }
           }
           ```

      6. bool用来做复合查询

         复合语句可以合并，任何其他查询语句，包括符合语句。这也就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

         - must：必须达到must所列举的所有条件
         - must_not：必须不匹配must_not所列举的所有条件。
         - should：应该满足should所列举的条件。满足条件最好，不满足也可以，**满足得分更高**

         实例：查询gender=m，并且address=mill的数据

         ```json
         GET bank/_search
         {
            "query":{
                 "bool":{
                      "must":[
                       {"match":{"address":"mill"}},
                       {"match":{"gender":"M"}}
                      ]
                  }
             }
         }
         ```

         **must_not：必须不是指定的情况**

         ```json
         GET bank/_search
         {
           "query": {
             "bool": {
               "must": [
                 { "match": { "gender": "M" }},
                 { "match": {"address": "mill"}}
               ],
               "must_not": [
                 { "match": { "age": "38" }}
               ]
            }
         }
         ```

         **should：应该达到should列举的条件，如果到达会增加相关文档的评分，并不会改变查询的结果。如果query中只有should且只有一种匹配规则，那么should的条件就会被作为默认匹配条件而去改变查询结果。**

         实例：匹配lastName应该等于Wallace的数据

         ```json
         GET bank/_search
         {
           "query": {
             "bool": {
               "must": [
                 {
                   "match": {
                     "gender": "M"
                   }
                 },
                 {
                   "match": {
                     "address": "mill"
                   }
                 }
               ],
               "must_not": [
                 {
                   "match": {
                     "age": "28"
                   }
                 }
               ],
               "should": [
                 {
                   "match": {
                     "lastname": "Wallace"
                   }
                 }
               ]
             }
           }
         }
         ```

         能够看到相关度越高，得分也越高。

      7. Filter【结果过滤】

         并不是所有的查询都需要产生分数，特别是哪些仅用于filtering过滤的文档。为了不计算分数，elasticsearch会自动检查场景并且优化查询的执行。

         ```json
         GET bank/_search
         {
           "query": {
             "bool": {
               "must": [
                 { "match": {"address": "mill" } }
               ],
               "filter": {  # query.bool.filter
                 "range": {
                   "age": {
                     "gte": "18",
                     "lte": "30"
                   }
                 }
               }
             }
           }
         }
         ```

         这里先是查询所有匹配address=mill的文档，然后再根据18<=age<=30进行过滤查询结果

         > 在boolean查询中，must, should 和must_not 元素都被称为查询子句 。 文档是否符合每个“must”或“should”子句中的标准，决定了文档的“相关性得分”。 得分越高，文档越符合您的搜索条件。 默认情况下，Elasticsearch返回根据这些相关性得分排序的文档。

         > `“must_not”子句中的条件被视为“过滤器”。` 它影响文档是否包含在结果中， 但不影响文档的评分方式。 还可以显式地指定任意过滤器来包含或排除基于结构化数据的文档。

      8. term

         和match一样。匹配某个属性额值。全文检索字段用match，其他非text字段匹配用term

         > 字段.keyword：要精确匹配到（全部相等）

         ```json
         GET bank/_search
         {
           "query": {
             "term": {
               "address": "mill Road"
             }
           }
         }
         ```

         ```json
         {
           "took" : 0,
           "timed_out" : false,
           "_shards" : {
             "total" : 1,
             "successful" : 1,
             "skipped" : 0,
             "failed" : 0
           },
           "hits" : {
             "total" : {
               "value" : 0, 没有
               "relation" : "eq"
             },
             "max_score" : null,
             "hits" : [ ]
           }
         }
         ```

      9. Aggregation（聚合）

         [API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/search-aggregations.html)

         聚合提供了从数据中分组和提取数据的能力。最简单的聚合方法大致等于SQL Group by和SQL聚合函数。在elasticsearch中，执行搜索返回this（命中结果），并且同时返回聚合结果，把以响应中的所有hits（命中结果）分隔开的能力。这是非常强大且有效的，你可以执行查询和多个聚合，并且在一次使用中得到各自的（任何一个的）返回结果，使用一次简洁和简化的API啦避免网络往返。

         aggs：执行聚合。聚合语法如下：

         ```json
         "aggs":{ # 聚合
             "aggs_name这次聚合的名字，方便展示在结果集中":{
                 "AGG_TYPE聚合的类型(avg,term,terms)":{}
              }
         }
         ```

         - terms：看值的可能性分布
         - avg：看值的分布平均

         例：**搜索address中包含mill的所有人的年龄分布以及平均年龄，但不显示这些人的详情**

         ```json
         # 分别为包含mill、，平均年龄、
         GET bank/_search
         {
           "query": { # 查询出包含mill的
             "match": {
               "address": "mill"
             }
           },
           "aggs": { #基于查询聚合
             "ageAgg": {  # 聚合的名字，随便起
               "terms": { # 看值的可能性分布
                 "field": "age",
                 "size": 10
               }
             },
             "ageAvg": { 
               "avg": { # 看age值的平均
                 "field": "age"
               }
             },
             "balanceAvg": {
               "avg": { # 看balance的平均
                 "field": "balance"
               }
             }
           },
           "size": 0  # 不看详情
         }
         ```

         查询结果：

         ```json
         {
           "took" : 2,
           "timed_out" : false,
           "_shards" : {
             "total" : 1,
             "successful" : 1,
             "skipped" : 0,
             "failed" : 0
           },
           "hits" : {
             "total" : {
               "value" : 4, // 命中4条
               "relation" : "eq"
             },
             "max_score" : null,
             "hits" : [ ]
           },
           "aggregations" : {
             "ageAgg" : { // 第一个聚合的结果
               "doc_count_error_upper_bound" : 0,
               "sum_other_doc_count" : 0,
               "buckets" : [
                 {
                   "key" : 38,
                   "doc_count" : 2
                 },
                 {
                   "key" : 28,
                   "doc_count" : 1
                 },
                 {
                   "key" : 32,
                   "doc_count" : 1
                 }
               ]
             },
             "ageAvg" : { // 第二个聚合的结果
               "value" : 34.0
             },
             "balanceAvg" : {
               "value" : 25208.0
             }
           }
         }
         ```

         ###### 子聚合

         复杂：按照年龄聚合，并且求这些年龄段的这些人的平均薪资

         > 写到一个聚合里是基于上个聚合进行子聚合。
         >
         > 下面求每个age分布的平均balance

         ```json
         GET bank/_search
         {
           "query": {
             "match_all": {}
           },
           "aggs": {
             "ageAgg": {
               "terms": { # 看分布
                 "field": "age",
                 "size": 100
               },
               "aggs": { # 与terms并列
                 "ageAvg": { #平均
                   "avg": {
                     "field": "balance"
                   }
                 }
               }
             }
           },
           "size": 0
         }
         ```

         输出结果：

         ```json
         {
           "took" : 49,
           "timed_out" : false,
           "_shards" : {
             "total" : 1,
             "successful" : 1,
             "skipped" : 0,
             "failed" : 0
           },
           "hits" : {
             "total" : {
               "value" : 1000,
               "relation" : "eq"
             },
             "max_score" : null,
             "hits" : [ ]
           },
           "aggregations" : {
             "ageAgg" : {
               "doc_count_error_upper_bound" : 0,
               "sum_other_doc_count" : 0,
               "buckets" : [
                 {
                   "key" : 31,
                   "doc_count" : 61,
                   "ageAvg" : {
                     "value" : 28312.918032786885
                   }
                 },
                 {
                   "key" : 39,
                   "doc_count" : 60,
                   "ageAvg" : {
                     "value" : 25269.583333333332
                   }
                 },
                 {
                   "key" : 26,
                   "doc_count" : 59,
                   "ageAvg" : {
                     "value" : 23194.813559322032
                   }
                 },
                 {
                   "key" : 32,
                   "doc_count" : 52,
                   "ageAvg" : {
                     "value" : 23951.346153846152
                   }
                 },
                 {
                   "key" : 35,
                   "doc_count" : 52,
                   "ageAvg" : {
                     "value" : 22136.69230769231
                   }
                 },
                 {
                   "key" : 36,
                   "doc_count" : 52,
                   "ageAvg" : {
                     "value" : 22174.71153846154
                   }
                 },
                 {
                   "key" : 22,
                   "doc_count" : 51,
                   "ageAvg" : {
                     "value" : 24731.07843137255
                   }
                 },
                 {
                   "key" : 28,
                   "doc_count" : 51,
                   "ageAvg" : {
                     "value" : 28273.882352941175
                   }
                 },
                 {
                   "key" : 33,
                   "doc_count" : 50,
                   "ageAvg" : {
                     "value" : 25093.94
                   }
                 },
                 {
                   "key" : 34,
                   "doc_count" : 49,
                   "ageAvg" : {
                     "value" : 26809.95918367347
                   }
                 },
                 {
                   "key" : 30,
                   "doc_count" : 47,
                   "ageAvg" : {
                     "value" : 22841.106382978724
                   }
                 },
                 {
                   "key" : 21,
                   "doc_count" : 46,
                   "ageAvg" : {
                     "value" : 26981.434782608696
                   }
                 },
                 {
                   "key" : 40,
                   "doc_count" : 45,
                   "ageAvg" : {
                     "value" : 27183.17777777778
                   }
                 },
                 {
                   "key" : 20,
                   "doc_count" : 44,
                   "ageAvg" : {
                     "value" : 27741.227272727272
                   }
                 },
                 {
                   "key" : 23,
                   "doc_count" : 42,
                   "ageAvg" : {
                     "value" : 27314.214285714286
                   }
                 },
                 {
                   "key" : 24,
                   "doc_count" : 42,
                   "ageAvg" : {
                     "value" : 28519.04761904762
                   }
                 },
                 {
                   "key" : 25,
                   "doc_count" : 42,
                   "ageAvg" : {
                     "value" : 27445.214285714286
                   }
                 },
                 {
                   "key" : 37,
                   "doc_count" : 42,
                   "ageAvg" : {
                     "value" : 27022.261904761905
                   }
                 },
                 {
                   "key" : 27,
                   "doc_count" : 39,
                   "ageAvg" : {
                     "value" : 21471.871794871793
                   }
                 },
                 {
                   "key" : 38,
                   "doc_count" : 39,
                   "ageAvg" : {
                     "value" : 26187.17948717949
                   }
                 },
                 {
                   "key" : 29,
                   "doc_count" : 35,
                   "ageAvg" : {
                     "value" : 29483.14285714286
                   }
                 }
               ]
             }
           }
         }
         ```

         复杂子聚合：查出所有年龄分布，并且这些**年龄段**中M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资

         ```json
         GET bank/_search
         {
           "query": {
             "match_all": {}
           },
           "aggs": {
             "ageAgg": {
               "terms": {  #  看age分布
                 "field": "age",
                 "size": 100
               },
               "aggs": { # 子聚合
                 "genderAgg": {
                   "terms": { # 看gender分布
                     "field": "gender.keyword" # 注意这里，文本字段应该用.keyword
                   },
                   "aggs": { # 子聚合
                     "balanceAvg": {
                       "avg": { # 男女性的平均
                         "field": "balance"
                       }
                     }
                   }
                 },
                 "ageBalanceAvg": {
                   "avg": { #age分布的平均（男女）
                     "field": "balance"
                   }
                 }
               }
             }
           },
           "size": 0
         }
         ```

         输出结果：

         ```json
         {
           "took" : 119,
           "timed_out" : false,
           "_shards" : {
             "total" : 1,
             "successful" : 1,
             "skipped" : 0,
             "failed" : 0
           },
           "hits" : {
             "total" : {
               "value" : 1000,
               "relation" : "eq"
             },
             "max_score" : null,
             "hits" : [ ]
           },
           "aggregations" : {
             "ageAgg" : {
               "doc_count_error_upper_bound" : 0,
               "sum_other_doc_count" : 0,
               "buckets" : [
                 {
                   "key" : 31,
                   "doc_count" : 61,
                   "genderAgg" : {
                     "doc_count_error_upper_bound" : 0,
                     "sum_other_doc_count" : 0,
                     "buckets" : [
                       {
                         "key" : "M",
                         "doc_count" : 35,
                         "balanceAvg" : {
                           "value" : 29565.628571428573
                         }
                       },
                       {
                         "key" : "F",
                         "doc_count" : 26,
                         "balanceAvg" : {
                           "value" : 26626.576923076922
                         }
                       }
                     ]
                   },
                   "ageBalanceAvg" : {
                     "value" : 28312.918032786885
                   }
                 }
               ]
                 .......//省略其他
             }
           }
         }
         
         ```

         nested（嵌套）对象聚合

         ```json
         GET articles/_search
         {
           "size": 0, 
           "aggs": {
             "nested": {
               "nested": {
                 "path": "payment"
               },
               "aggs": {
                 "amount_avg": {
                   "avg": {
                     "field": "payment.amount"
                   }
                 }
               }
             }
           }
         }
         ```

   3. Mapping

      映射定义文档如何被存储检索的

      1. 字段类型

         1. 核心类型

            1. 字符串

               - `text`⽤于全⽂索引，搜索时会自动使用分词器进⾏分词再匹配
               - `keyword` 不分词，搜索时需要匹配完整的值

            2. 数值型

               - 整形： byte，short，integer，long
               - 浮点型： float, half_float, scaled_float，double

            3. 日期型：date

            4. 范围型

               - integer_range， long_range， float_range，double_range，date_range

                 gt是大于，lt是小于，e是equals等于。

                 age_limit的区间包含了此值的文档都算是匹配。

            5. 布尔：boolean

            6. ⼆进制：binary 会把值当做经过 base64 编码的字符串，默认不存储，且不可搜索

         2. 复合类型

            1. 对象：object一个对象中可以嵌套对象
            2. 数组：array
            3. 嵌套类型：nested 用于json对象数组

         3. 地理类型

            1. 地理坐标：geo_point用于描述经纬度坐标
            2. 地理图形：geo_shape用于描述复杂形状，如多边形

         4. 特定类型

      2. 映射

         Maping是用来定义一个文档（document），以及它所包含的属性（field）是如何存储和索引的。比如：使用maping来定义：

         - 哪些字符串属性应该被看做全文本属性（full text fields）；

         - 哪些属性包含数字，日期或地理位置；

         - 文档中的所有属性是否都嫩被索引（all 配置）；

         - 日期的格式；

         - 自定义映射规则来执行动态添加属性；

         - 查看mapping信息：GET bank/_mapping

           ```json
           {
             "bank" : {
               "mappings" : {
                 "properties" : {
                   "account_number" : {
                     "type" : "long" # long类型
                   },
                   "address" : {
                     "type" : "text", # 文本类型，会进行全文检索，进行分词
                     "fields" : {
                       "keyword" : { # addrss.keyword
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "age" : {
                     "type" : "long"
                   },
                   "balance" : {
                     "type" : "long"
                   },
                   "city" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "email" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "employer" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "firstname" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "gender" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "lastname" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   },
                   "state" : {
                     "type" : "text",
                     "fields" : {
                       "keyword" : {
                         "type" : "keyword",
                         "ignore_above" : 256
                       }
                     }
                   }
                 }
               }
             }
           }
           ```

           **创建映射PUT /my_index**]

           > 第一次存储数据的时候es就猜出了映射
           >
           > 第一次存储数据前可以指定映射

           创建索引并指定映射

           ```json
           PUT /my_index
           {
             "mappings": {
               "properties": {
                 "age": {
                   "type": "integer"
                 },
                 "email": {
                   "type": "keyword" # 指定为keyword
                 },
                 "name": {
                   "type": "text" # 全文检索。保存时候分词，检索时候进行分词匹配
                 }
               }
             }
           }
           ```

           输出：

           ```json
           {
             "acknowledged" : true,
             "shards_acknowledged" : true,
             "index" : "my_index"
           }
           ```

           **查看映射GET /my_index**

           ```json
           GET /my_index
           ```

           输出结果：

           ```json
           {
             "my_index" : {
               "aliases" : { },
               "mappings" : {
                 "properties" : {
                   "age" : {
                     "type" : "integer"
                   },
                   "email" : {
                     "type" : "keyword"
                   },
                   "employee-id" : {
                     "type" : "keyword",
                     "index" : false
                   },
                   "name" : {
                     "type" : "text"
                   }
                 }
               },
               "settings" : {
                 "index" : {
                   "creation_date" : "1588410780774",
                   "number_of_shards" : "1",
                   "number_of_replicas" : "1",
                   "uuid" : "ua0lXhtkQCOmn7Kh3iUu0w",
                   "version" : {
                     "created" : "7060299"
                   },
                   "provided_name" : "my_index"
                 }
               }
             }
           }
           ```

           **添加新的字段映射/my_index/_mapping**

           ```json
           PUT /my_index/_mapping
           {
             "properties": {
               "employee-id": {
                 "type": "keyword",
                 "index": false # 字段不能被检索。检索
               }
             }
           }
           ```

           这里的 “index”: false，表明新增的字段不能被检索，只是一个冗余字段。

           ###### 不能更新映射

           对于已经存在的字段映射，我们不能更新。更新必须创建新的索引，进行数据迁移。

           ###### 数据迁移

           先创建new_twitter的正确映射。

           然后使用如下方式进行数据迁移。

           ```json
           6.0以后写法
           POST reindex
           {
             "source":{
                 "index":"twitter"
              },
             "dest":{
                 "index":"new_twitters"
              }
           }
           
           
           老版本写法
           POST reindex
           {
             "source":{
                 "index":"twitter",
                 "twitter":"twitter"
              },
             "dest":{
                 "index":"new_twitters"
              }
           }
           ```

      3. 分词

         一个tokenizer（分词器）接收一个字符流，将之分割为独立的tokens（词元，通常是独立的单词），然后输出tokens流。

         例如：whitespace tokenizer遇到空白字符时分割文本。它会将文本"Quick brown fox!"分割为[Quick,brown,fox!]

         该tokenizer（分词器）还负责记录各个terms(词条)的顺序或position位置（用于phrase短语和word proximity词近邻查询），以及term（词条）所代表的原始word（单词）的start（起始）和end（结束）的character offsets（字符串偏移量）（用于高亮显示搜索的内容）。

         elasticsearch提供了很多内置的分词器（标准分词器），可以用来构建custom analyzers（自定义分词器）。

         关于分词器： https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis.html

         ```json
         POST _analyze
         {
           "analyzer": "standard",
           "text": "The 2 Brown-Foxes bone."
         }
         ```

         执行结果：

         ```json
         {
           "tokens" : [
             {
               "token" : "the",
               "start_offset" : 0,
               "end_offset" : 3,
               "type" : "<ALPHANUM>",
               "position" : 0
             },
             {
               "token" : "2",
               "start_offset" : 4,
               "end_offset" : 5,
               "type" : "<NUM>",
               "position" : 1
             },
             {
               "token" : "brown",
               "start_offset" : 6,
               "end_offset" : 11,
               "type" : "<ALPHANUM>",
               "position" : 2
             },
             {
               "token" : "foxes",
               "start_offset" : 12,
               "end_offset" : 17,
               "type" : "<ALPHANUM>",
               "position" : 3
             },
             {
               "token" : "bone",
               "start_offset" : 18,
               "end_offset" : 22,
               "type" : "<ALPHANUM>",
               "position" : 4
             }
           ]
         }
         ```

         对于中文，我们需要安装额外的分词器

         1. 安装ik分词器

            所有的语言分词，默认使用的都是“Standard Analyzer”，但是这些分词器针对于中文的分词，并不友好。为此需要安装中文的分词器。

            注意：不能用默认elasticsearch-plugin install xxx.zip 进行自动安装
            https://github.com/medcl/elasticsearch-analysis-ik/releases

            在前面安装的elasticsearch时，我们已经将elasticsearch容器的“/usr/share/elasticsearch/plugins”目录，映射到宿主机的“ /mydata/elasticsearch/plugins”目录下，所以比较方便的做法就是下载“/elasticsearch-analysis-ik-7.4.2.zip”文件，然后解压到该文件夹下即可。安装完毕后，需要重启elasticsearch容器。

            如果不嫌麻烦，还可以采用如下的方式：

            1. **查看elasticsearch版本号：**

               ```json
               [vagrant@localhost ~]$ curl http://localhost:9200
               {
                 "name" : "66718a266132",
                 "cluster_name" : "elasticsearch",
                 "cluster_uuid" : "xhDnsLynQ3WyRdYmQk5xhQ",
                 "version" : {
                   "number" : "7.4.2",
                   "build_flavor" : "default",
                   "build_type" : "docker",
                   "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
                   "build_date" : "2019-10-28T20:40:44.881551Z",
                   "build_snapshot" : false,
                   "lucene_version" : "8.2.0",
                   "minimum_wire_compatibility_version" : "6.8.0",
                   "minimum_index_compatibility_version" : "6.0.0-beta1"
                 },
                 "tagline" : "You Know, for Search"
               }
               ```

            2. **进入es容器内部plugin目录**

               - docker exec -it 容器id /bin/bash

               ```json
               [vagrant@localhost ~]$ sudo docker exec -it elasticsearch /bin/bash
               
               [root@66718a266132 elasticsearch]# pwd
               /usr/share/elasticsearch
               [root@66718a266132 elasticsearch]# pwd
               /usr/share/elasticsearch
               [root@66718a266132 elasticsearch]# yum install wget
               #下载ik7.4.2
               [root@66718a266132 elasticsearch]# wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.4.2/elasticsearch-analysis-ik-7.4.2.zip
               ```

               - unzip 下载的文件

               ```
               yum install -y unzip zip
               [root@66718a266132 elasticsearch]# unzip elasticsearch-analysis-ik-7.4.2.zip -d ik
               
               #移动到plugins目录下
               [root@66718a266132 elasticsearch]#   
               
               chmod -R 777 plugins/ik
               
               docker restart elasticsearch
               
               ```

               - rm -rf *.zip

               ```json
               [root@66718a266132 elasticsearch]# rm -rf elasticsearch-analysis-ik-7.6.2.zip 
               ```

               - 查看是否安装成功

               ```
               elasticsearch-plugin list
               ```

         2. 测试ik分词器

            使用默认分词器

            ```json
            GET _analyze
            {
               "text":"我是中国人"
            }
            
            ```

            结果：

            ```json
            {
              "tokens" : [
                {
                  "token" : "我",
                  "start_offset" : 0,
                  "end_offset" : 1,
                  "type" : "<IDEOGRAPHIC>",
                  "position" : 0
                },
                {
                  "token" : "是",
                  "start_offset" : 1,
                  "end_offset" : 2,
                  "type" : "<IDEOGRAPHIC>",
                  "position" : 1
                },
                {
                  "token" : "中",
                  "start_offset" : 2,
                  "end_offset" : 3,
                  "type" : "<IDEOGRAPHIC>",
                  "position" : 2
                },
                {
                  "token" : "国",
                  "start_offset" : 3,
                  "end_offset" : 4,
                  "type" : "<IDEOGRAPHIC>",
                  "position" : 3
                },
                {
                  "token" : "人",
                  "start_offset" : 4,
                  "end_offset" : 5,
                  "type" : "<IDEOGRAPHIC>",
                  "position" : 4
                }
              ]
            }
            
            ```

            ```json
            GET _analyze
            {
               "analyzer": "ik_smart", 
               "text":"我是中国人"
            }
            ```

            结果：

            ```json
            {
              "tokens" : [
                {
                  "token" : "我",
                  "start_offset" : 0,
                  "end_offset" : 1,
                  "type" : "CN_CHAR",
                  "position" : 0
                },
                {
                  "token" : "是",
                  "start_offset" : 1,
                  "end_offset" : 2,
                  "type" : "CN_CHAR",
                  "position" : 1
                },
                {
                  "token" : "中国人",
                  "start_offset" : 2,
                  "end_offset" : 5,
                  "type" : "CN_WORD",
                  "position" : 2
                }
              ]
            }
            ```

            ```json
            GET _analyze
            {
               "analyzer": "ik_max_word", 
               "text":"我是中国人"
            }
            
            ```

            结果：

            ```json
            {
              "tokens" : [
                {
                  "token" : "我",
                  "start_offset" : 0,
                  "end_offset" : 1,
                  "type" : "CN_CHAR",
                  "position" : 0
                },
                {
                  "token" : "是",
                  "start_offset" : 1,
                  "end_offset" : 2,
                  "type" : "CN_CHAR",
                  "position" : 1
                },
                {
                  "token" : "中国人",
                  "start_offset" : 2,
                  "end_offset" : 5,
                  "type" : "CN_WORD",
                  "position" : 2
                },
                {
                  "token" : "中国",
                  "start_offset" : 2,
                  "end_offset" : 4,
                  "type" : "CN_WORD",
                  "position" : 3
                },
                {
                  "token" : "国人",
                  "start_offset" : 3,
                  "end_offset" : 5,
                  "type" : "CN_WORD",
                  "position" : 4
                }
              ]
            }
            ```

         3. 自定义词库

            比如我们要把尚硅谷算作一个词

            - 修改/usr/share/elasticsearch/plugins/ik/config中的IKAnalyzer.cfg.xml

            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
            <properties>
            	<comment>IK Analyzer 扩展配置</comment>
            	<!--用户可以在这里配置自己的扩展字典 -->
            	<entry key="ext_dict"></entry>
            	 <!--用户可以在这里配置自己的扩展停止词字典-->
            	<entry key="ext_stopwords"></entry>
            	<!--用户可以在这里配置远程扩展字典 -->
            	<entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry> 
            	<!--用户可以在这里配置远程扩展停止词字典-->
            	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
            </properties>
            ```

            修改完成后，需要重启elasticsearch容器，否则修改不生效。docker restart elasticsearch

            更新完成后，es只会对于新增的数据用更新分词。历史数据是不会重新分词的。如果想要历史数据重新分词，需要执行：

            ```
            POST my_index/_update_by_query?conflicts=proceed
            ```

            - 安装nginx

              1. 随便启动一个nginx实例，只是为了复制出配置

                 ```
                 docker run -p80:80 --name nginx -d nginx:1.10   
                 ```

              2. 将容器内的配置文件拷贝到/mydata/nginx/conf/ 下

                 ```
                 [root@10 mydata]# docker container cp nginx:/etc/nginx .
                 [root@10 mydata]# ls
                 elasticsearch  mysql  nginx  redis
                 [root@10 mydata]# cd nginx/
                 [root@10 nginx]# ls
                 conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params  uwsgi_params  win-utf
                 [root@10 nginx]# cd ../
                 [root@10 mydata]# ls
                 elasticsearch  mysql  nginx  redis
                 [root@10 mydata]# mv nginx conf
                 [root@10 mydata]# ls
                 conf  elasticsearch  mysql  redis
                 [root@10 mydata]# mkdir nginx
                 [root@10 mydata]# mv conf nginx/
                 [root@10 mydata]# ls
                 elasticsearch  mysql  nginx  redis
                 [root@10 mydata]# cd nginx/
                 [root@10 nginx]# ls
                 conf
                 ```

              3. 终止原容器：

                 ```
                 docker stop nginx
                 ```

              4. 执行命令删除原容器：

                 ```
                 docker rm nginx
                 ```

              5. 创建新的Nginx，执行以下命令

                 ```
                 mkdir -p /mydata/nginx/html
                 mkdir -p /mydata/nginx/logs
                 docker run -p 80:80 --name nginx \
                  -v /mydata/nginx/html:/usr/share/nginx/html \
                  -v /mydata/nginx/logs:/var/log/nginx \
                  -v /mydata/nginx/conf/:/etc/nginx \
                  -d nginx:1.10
                 ```

              6. 设置开机启动nginx

                 ```
                 docker update nginx --restart=always
                 ```

              7. 创建“/mydata/nginx/html/index.html”文件，测试是否能够正常访问

                 ```
                 echo '<h2>hello nginx!</h2>' >index.html
                 ```

              8. 访问：http://ngix所在主机的IP:80/index.html

            - 安装好nginx后

              ```
              mkdir /mydata/nginx/html/es
              cd /mydata/nginx/html/es
              vim fenci.txt
              输入乔碧萝
              ```

              测试效果：

              ```
              GET _analyze
              {
                 "analyzer": "ik_max_word", 
                 "text":"乔碧萝殿下"
              }
              ```

              结果：

              ```
              {
                "tokens" : [
                  {
                    "token" : "乔碧萝",
                    "start_offset" : 0,
                    "end_offset" : 3,
                    "type" : "CN_WORD",
                    "position" : 0
                  },
                  {
                    "token" : "殿下",
                    "start_offset" : 3,
                    "end_offset" : 5,
                    "type" : "CN_WORD",
                    "position" : 1
                  }
                ]
              }
              ```

              
