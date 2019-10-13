# Redis

1. 关系型数据库(sql):

   - mysql,oracle

   - 特点:数据和数据之间,表和字段之间

   - 部门表 员工id:001,员工表 id : 001

     用户表  用户名,密码

     分类表和 商品表,一对多

   - 优点:1.数据之间有关系,进行数据的crud非常方便2.关系型数据库有事务操作,保证数据的完整性

   - 缺点:1.因为数据和数据之间有关系的,是由底层大量算法保证的,大量算法会拉低系统的运行速度,大量算法会消耗系统资源2.海量数据的crud时会显得无能为力.很可能宕机.3.海量数据环境下对数据表进行维护,也很不便

   - 适合处理一般量级数据,安全.

2. 非关系型数据库(nosql):为了处理海量数据,需要将关系型数据库的关系去掉,设计之初是为了替代关系型数据库

   - Redis
   - 优点:1.海量数据的增删改查,非常轻松应对2.海量数据的维护非常轻松2.是完全开源免费的,遵守BSD协议,是一个高性能(NoSQL)的key-value数据库
   - 缺点:1.  数据和数据之间没有关系,所以不能一目了然  2.  非关系型数据库,没有关系,没有强大的事务保证数据的完整和安全
   - 适合处理海量数据,效率,不一定安全.

3. 开发:关系型数据库+非关系型数据库共同支撑一个项目,重要的数据放在关系型数据库里,不重要的经常查询的放在关系型数据里

4. 使用环境:1.  作为关系型数据的缓存存在.  2.  任务队列  3.  大量数据运算(比如取两个大量数据的交集)  4.  排行榜

5. Redis安装;推荐安装在LInux上

   - 因为Redis是c语言开发的,所以需要提供gcc环境

     ```
     yum install gcc-c++
     ```

6. Redis数据类型

   - redis使用的是键值对保存数据(map).
   - key:全部都是字符串(key名自定义,不要过长否则影响使用效率,key查询时先从最短的开始查)
   - value:有五种数据类型(String,hash(和json类似),list(LinkedList链表添加删除效率高),set集合(hashset),有序的set集合)

7. redis命令-string命令

   字符串类型是redis中最为基础和常用的数据存储类型,字符串在redis中是**二进制安全**的,这便意味着该类型存入和获取的数据相同.在redis中字符串类型的value最多可以容纳的数据长度为512M.

   (

   ​	mysql:关系型数据库,二进制不安全,如果编码表不同会产生乱码

   ​	redis:非关系型数据库,二进制安全,服务器端不会进行编解码只在客户端编解码,只会传递二进制数据

   )

   - 赋值(更改)操作:
     - 语法:set key value;
     - 如果赋予同一个key不同的value,新value会覆盖老的value
   - 取值操作:
     - 语法:get key
     - 如果与该key进行关联的value不是String类型,redis会报错,因为get key命令只能用于获取String类型的value.如果key不存在,则返回nil
   - 删除操作:
     - 语法:del key
     - 返回值是数据类型,表示删了几条数据
   - 获取并赋值操作:
     - 语法:getset key value 
   - 数值增减操作:
     - 语法:incr key
     - 将指定的key的value原子性的递增1,如果该key不存在,其初始值为0,在incr之后其值为1,如果value的值不能转换成整形,如hello,该操作将执行失败并返回相应的错误信息(常用作计数器)
     - 语法:decr key
     - 将指定的key的value原子性的递减1,如果该key不存在,其初始值为0,在incr之后其值为-1,如果value的值不能转换成整形,如hello,该操作将执行失败并返回相应的错误信息(常用作计数器)
   - 拼接字符串:
     - 语法:append key value
     - 拼凑字符串,如果该key存在,则在原有的value后追加该值,如果不存在则重新创建一个key value
   - 将数字值自增任意值
     - 语法:incrby key increment
     - 将指定的key的value原子性的递增increment,如果该key不存在,其初始值为0,在incr之后其值为increment,如果value的值不能转换成整形,如hello,该操作将执行失败并返回相应的错误信息(常用作计数器)
   - 将数字值自减任意值
     - 语法:decrby key decrement
     - 将指定的key的value原子性的递减increment,如果该key不存在,其初始值为0,在incr之后其值为-increment,如果value的值不能转换成整形,如hello,该操作将执行失败并返回相应的错误信息(常用作计数器)
   - String使用环境:
     - 主要用于保存json格式的字符串
     - 扩展:flushdb命令,会刷新数据库删除所有的键值对

8. redis命令-hash

   Redis中的hsh类型可以看成具有String key 和String value的map容器,所以该类型非常适合存储值对象的信息.hash--->{username:"张三",age:"18",sex:"男"}------>javabean

   - 特点:占用的磁盘空间极少存储同一个javabeanhash要比String占用的磁盘空间少很多
   - 赋值操作
     - 语法:hset key field value
     - 为指定的key设定field/value对(键值对)
     - 语法:hmset key field value field2 value2 ...
     - 设置key中的多个键值对
   - 取值操作
     - 语法:hget key field
     - 返回指定key 中的filed的值
     - 语法:hmget key field field2
     - 返回指定key中的多个field的值
     - 语法:hgetall key
     - 返回指定key中的所有field value
   - 删除操作
     - 语法:hel key field field2
     - 可以删除一个或多个字段,返回值是删除的字段个数
     - 语法:del key
     - 删除整个hash
   - 增加数字操作
     - 语法:hincrby key filed increment
     - 设置key中field的值增加increment
   - 自学命令
     - 语法:hexists key field
     - 判断指定的key中的field是否存在
     - 语法:hlen key
     - 返回指定的key中包含多少个字段
     - 语法:hkeys key
     - 获得所有字段
     - 语法:hvals key
     - 获得所有的value

9. redis-LinkedList命令

   java List: 数组ArrayList(有索引查询快) 链表 LinkedList(无索引查询慢,插入删除块)

   为什么redis选取了链表?redis做的操作最多的是进行元素的增删

   - 使用环境:1.  做大数据集合的增删.  2.  任务队列

   - 赋值操作:
   
     - 语法:lpush key value1 value2 ...                           
     - 为指定的key从左侧插入value值
   
     - 语法:rpush key value value2 ...
     - 为指定的key从右侧插入value值 
   
   - 查询操作:
   
     - 语法:lrange key start end
     - 获取链表中从start到end元素的值,start,end从0开始计数,也可以为负数,若为-1,则表示链表尾部的元素,-2表示倒数第二个,依次类推
   
   - 删除操作:
   
     - 语法:lpop key
     - 返回弹出指定的key关联的链表中的第一个元素,即头部元素,如果该key不存在返回nil,若key存在返回链表的头部元素
     - 语法:rpop key
     - 从尾部弹出元素
   
   - 扩展命令
   
     - 语法:llen key
     - 返回指定key关联的链表中的元素个数
     - 语法:lrem key count value
     - 删除count个值为value的元素,如果count大于0则从头向尾遍历并删除count个值为value的元素,如果count小于0则从尾向头遍历并删除,如果count等于0,则删除链表中所有等于value的元素
     - 语法:lset key index value 
     - 设置链表中的index的角标的元素值,0代表链表的头元素,-1代表元素的尾元素,操作的角标不存在则抛出异常
     - 语法:linset key before|after pivot value
     - 在pivot元素前或者后插入value这个元素
     - **语法:rpoplpush resource destination**
     - 将resource链表中的尾部元素弹出并添加到destination链表头部.(循环操作,任务队列),如果要做循环队列,只需要rpoplpush 操作同一个链表即可
   
10. redis-set

    java中hashset 无序且不重复,redis操作中,涉及到两个大数据集合的并集,交集,差集运算

    - 赋值操作
      - 语法:**sadd key value value2....**
      - 向set中添加一个或多个数据,如果指定key中已有value元素,则不会重复添加,返回成功添加的条数
    - 取值操作
      - 语法:**smembers key**
      - 获取指定key的set集合中所有成员
      - 语法:**sismember key member**
      - 判断参数中指定的成员是否存在该set中,1表示存在,0表示不存在,或者key本身就不存在(速度极快)
    - 删除操作
      - 语法:**srem key members  member2**
      - 删除一个或多个指定key的set集合中的member元素
    - 差集操作
      - 语法:sdiff key1 key2 ...
      - 返回指定key1与key2中相差的成员,而且与**key的顺序有关**,即返回差集,语义就是属于key1但不属于key2
    - 交集操作
      - 语法:sinter key1 key2 ...
      - 返回多个key指定set集合的交集
    - 并集操作
      - 语法:sunion key1 key2 ...
      - 返回并集
    - 扩展命令
      - **语法:scard key**
      - **获取key指定的set集合中元素的数量**
      - **语法:srandmember key**
      - **随机返回key指定的set集合中的一个元素**
      - **语法:sdiffstore destination key1 key2**
      - **将key指定的set集合做差集并保存到destination上**
      - **语法:sinterstore destination key1 key2**
      - **将key指定的set集合做交集并保存到destination上**
      - **语法:sunionstore destination key1 key2**
      - **将key指定的set集合做并集并保存到destination上**

11. redis-有序set(排行榜)

    - 赋值操作
      - 语法:zadd key score member score2 member2
      - 将所有成员以及该成员的分数存放到sorted-set中,如果该成员已存在,则会用新的分数替换原有的分数,返回值是新加入到集合中的成员个数,不包含之前以及存在的元素(默认排序由小到大)
    - 查询操作
      - 语法:zscore key member
      - 返回指定集合中的成员分数
      - 语法:zcard key
      - 返回集合中成员的数量
    - 删除操作
      - 语法:zrem key member
      - 删除集合中指定的成员,一个或多个
      - 语法:zcard key
      - 删除集合中的全部成员
    - 范围查询操作
      - 语法:**zrange key start end [withscores]**
      - 获取集合中角标为start-end的成员,withscores参数表示返回成员的分数
      - 语法:**zrevrange key start end [withscores]**
      - 倒序获取集合中角标为start-end的成员,withscores参数表示返回成员的分数
    - 范围删除操作
      - 语法:zremrangebyrank key start stop
      - 按照排名范围删除,start stop代表成员索引
      - 语法:zremrangebyscore key min max
      - 按照分数范围删除
    - 扩展命令
      - 语法:zrangebyscore key min max [withscores] limit start num
      - 返回分数范围内的从start开始的num个成员
      - 语法:zincrby key increment member
      - 设置指定成员的增加的分数,返回的是增加后的分数
      - 语法:zcount key min max
      - 返回分数范围内的成员数量
      - 语法:zrank key member
      - 返回成员在集合中的排名索引(正序)
      - 语法:zrevrank key member
      - 返回成员在集合中的排名索引(倒序)

12. 通用Redis命令

    redis有五种数据类型,String,hash,list,set,有序set

    - 语法keys pattern:
      - 查询key长度为4位的key名
        - keys ????
      - 查询所有key名
        - keys *
      - 查询key名中包含name的key
        - keys * name*
    - 语法del key1 key2..
      - 删除name1和name2两个key
        - del name1 name2'
    - 语法exists key
      - 判断name1是否存在
        - exists name
    - 语法:rename key newkey
      - 为name重命名name1
        - rename name name1
    - 语法:type key
      - 获取key的值的数据类型(五种)
    - 语法:expire key
      - 设置key生存时间,单位秒(默认永久存在)
    - 语法:ttl key
      - 获取该key剩余的生存时长,单位秒,如果为-1则为永久存在,如果 为0,redis会直接将key删除

13. 扩展知识-消息订阅与发布

    - 消息订阅
      - 语法:subscribe channel
        - 订阅频道
      - 语法:psubscribe channel *
        - 批量订阅频道(订阅以s开头的频道:psubscribe s*)

    - 消息发布
      - publish channel content
        - 在频道channel发布content消息

14. 扩展知识-多数据库

    - MySQL数据库可以自己用语句自定义创建
    - redis也是有数据库的,redis已经提前创建好了,默认有16个数据库,0...15,在redis上所做的所有数据操作,默认在0号数据中操作的,数据库和数据库之间不能共享键值对
    - 切换数据库
      - 语法:select 数据库名
    - 把某个键值对进行数据库移植
      - 语法:move key 数据库名
    - 当前数据库的清空
      - 语法:flushdb
    - 清空redis服务器的数据
      - 语法:flushall

15. 扩展知识-redis**批量操作**-事务

    - mysql-事务:目的是为了保证数据完整性和安全
    - redis-事务:目的为了进行redis语句的批量化执行
      - multi:开启事务用于标记事务的开始,之后执行的命令都将被存入命令队列,直到执行exec时这些命令才会被原子的执行,类似于关系型数据库中的begin transaction
      - exec:提交事务,类似于关系型数据库中的commit
      - discard:事务回滚,类似于关系型数据库中的rollback

16. 服务器命令

    - ping命令
      - 测试连接是否存活
    - echo命令
      - 在命令行窗口打印一些内容
    - select命令
      - 选择切换数据库
    - quit命令
      - 退出客户端类似ctrl+c
    - dbsize命令
      - 返回当前数据库中key的数目
    - info命令
      - 查看redis配置信息

17. 扩展知识-redis持久化

    - 关系型数据库持久化:任何增删改语句,都是在硬盘上做操作,断电后,因公安的数据还是存在
    - 非关系型数据库redis持久化:默认情况下,所有的增删改,数据都是在内存中进行操作的,断电以后内存中的数据会丢失,redis的部分数据会丢失,丢失的数据是保存在内存中的数据
    - redis持久化操作
      - RDB持久化策略:默认持久化操作
        - RDB相当于照快照(随时随地启动,会占用一部分系统资源),保存的是一种状态,优点是:快照保存数据速度极快,还原数据速度极快,适用于灾难备份 .缺点是:RDB机制符合要求就会照快照,服务器正常关闭时照快照,key满足一定条件,照快照,小内存机器不适合使用
        - RDB什么时候进行照快照
          - 服务器正常关闭时 ./bin/redis-cli shutdown
          - key满足一定条件 
            - save 900 1 (每900秒至少有一个key发生变化)
            - save 300 10 ( 每300秒至少有10个key发生变化)
            - save 60 10000 (每30秒)至少有10000个key发生变化
      - AOF持久化策略
        - 使用日志功能保存数据操作,适用于内存比较小的计算机
        - 默认AOF机制是关闭的
          - 不同步:不进行任何持久化操作
          - 每秒同步:每秒进行一次AOF保存数据
          - 每修改同步:只要进行key的变化语句,就进行AOF保存数据
        - AOF操作:
          - 只会保存导致key变化的语句
        - AOF配置
          - 开启AOF策略 :更改配置文件appendonly yes
          - always:每次有数据修改发生时都会写入AOF文件
          - everysec:每秒钟同步一次.该策略为AOF的缺省策略
          - no:从不同步 
        - 优点:持续性占用极少量的内存资源
        - 缺点:日志文件会特别大

18. jedis(java操作redis数据库技术)

    redis有什么命令,jedis就有什么方法

    - 单实例连接对象

      ```java
      @Test
          public void test(){
              Jedis jedis = new Jedis("192.168.243.128", 6379);
              jedis.set("name","张三");
              System.out.println(jedis.get("name"));
      
          }
      ```

      

    - 连接池连接

      ```java
      @Test
          public void test2(){
              //设置连接池的配置对象
              JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
              //设置池中最大连接数
              jedisPoolConfig.setMaxTotal(50);
              //设置空闲时间池中保有的最大连接数
              jedisPoolConfig.setMaxIdle(10);
              //设置连接池对象
              JedisPool pool = new JedisPool(jedisPoolConfig, "192.168.243.128", 6379);
              //从池中获取连接对象
              Jedis jedis = pool.getResource();
              System.out.println(jedis.get("name"));
              //连接归还给池中
              jedis.close();
      
          }
      ```

      

    - 连接池连接工具类

      ```java
      public class jedisUtil {
          //定义一个连接池对象
          private final static JedisPool POOL;
          //初始化POOL
          static {
              //设置连接池的配置对象
              JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
              //设置池中最大连接数
              jedisPoolConfig.setMaxTotal(50);
              //设置空闲时间池中保有的最大连接数
              jedisPoolConfig.setMaxIdle(10);
              //设置连接池对象
              POOL = new JedisPool(jedisPoolConfig, "192.168.243.128", 6379);
          }
          //从池中获取连接
          public static Jedis getJedis(){
              return POOL.getResource();
          }
      }
      
      ```

      

    

