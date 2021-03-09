# springboot+redis缓存

## 快速使用缓存

### 1.在启动类上加上开启缓存的注解：@EnableCaching

### 2.标识缓存注解

- @Cacheable

	- 1.方法运行前先去查询Cache组件，按照CacheName指定的名字获取（CacheManager先获取相应缓存）第一次获取缓存如果没有Cache组件会自动创建出来
	- 2.去Cache中查找缓存的内容，使用一个Key，默认就是方法的参数，key是按照某种策略生成的，默认是使用keyGenerator生成的，默认使用SimpleGenerator生成key，默认策略：如果没有参数：key=new SimpleKey();如果有一个参数:key=参数的值；如果有多个参数：key=new SimpleKey(params)；
	- 3.没有查到缓存就调用目标方法；
	- 4.将目标方法返回的结果，放进缓存中
	- 总结：@Cache标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，如果没有就运行方法并将结果放入缓存，以后再来调用就可以直接使用缓存中的数据。@Cacheable不能使用#result
	- 核心：使用CacheManager按照名字得到Cache组件，如果没有配置CacheManager则使用默认的。key使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key

- @CachePut

	- 即调用方法，又更新缓存
	- 1.先调用目标方法
	- 2.将目标方法的结果缓存起来
	- 3.指定的key需要与查询的key一样

- @CacheEvict

	- 缓存清除
	- key：指定要删除的数据
	- allEntries = true:删除指定cacheName缓存中所有数据
	- beforeInvocation = false:缓存的清除是否在方法之前执行，默认false代表方法执行之后执行，如果方法出现异常,缓存清除失败，如果设置为true在方法执行之前执行清除缓存，则缓存清除成功

- @Caching

	- 注解定义复杂的缓存规则
	- cacheable = {
                    @Cacheable(/*value = "emp",*/key = "#lastName")
            },
            put = {
                    @CachePut(/*value = "emp",*/key = "#result.id"),
                    @CachePut(/*value = "emp",*/key = "#result.email")
            }

## 整合Redis

### pom.xml引入redis之后容器保存的就是RedisCacheManager

- <!--引入redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

### 配置Redis

- #配置redis
spring.redis.host=192.168.1.145

### RedisCacheManager帮我们创建RedisCache来作为缓存主键，RedisCache是用来操作redis缓存数据的

### 默认保存数据k-v都是Object：利用序列化来保存到redis

### 为了缓存json到redis，需要自定义CacheManager

- @Primary//将某个缓存管理器设置为默认的
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //  解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                //设置缓存过期时间
                //.entryTtl(Duration.ofMinutes(150))
                //禁用缓存NULL值，不缓存NULL校验
                .disableCachingNullValues()
                //设置拼接前缀
                .computePrefixWith(cacheName -> cacheName)
                //设置自定义前缀
                //.prefixKeysWith("我的员工")
                // 设置CacheManager的值序列化方式为json序列化，可加入@Class属性
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer));
        // 使用RedisCacheConfiguration创建RedisCacheManager
        RedisCacheManager redisCacheManager = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
        return redisCacheManager;
    }

## 分支主题 3

*XMind: ZEN - Trial Version*