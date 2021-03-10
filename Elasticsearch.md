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

         

