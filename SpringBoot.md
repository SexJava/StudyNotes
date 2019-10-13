# SpringBoot

### 1. springboot父项目依赖(spring-boot-starter-parent)

方便以后导入依赖默认不用写版本号,但是只限于在父依赖中的dependencies里面管理的依赖

### 2. springboot场景启动器(spring-boot-starter-*)

帮忙导入对应场景模块的所需要的依赖,版本在父依赖中统一管理

### 3. 主程序类,主入口类

```java
@SpringBootApplication
public class SpringbootDemo1Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemo1Application.class, args);
    }

}
```

@**SpringBootApplication**:springboot应用标注在某个类上说明这个类是springboot的主配置类,springboot就应该运行在这个类的main方法来启动springboot应用;

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

```

- @**SpringBootConfiguration:springboot**配置类:

  标注在某个类上,表示这是个springboot的配置类

- @**Configuration**:配置类上来标注这个注解:

  配置类....配置文件:配置类也是容器中的一个组件:@component

- @**EnableAutoConfiguration**:开启自动配置功能;

  以前我们需要配置的东西,springboot帮我们自动配置

```java
@AutoConfigurationPackage@Import(AutoConfigurationImportSelector.class)public @interface EnableAutoConfiguration {
```

- @**AutoConfigurationPackage**:自动配置包,**将主配置类(@SpringBootApplication注解的类)的所在包及下面所有子包里面的所欲组件扫描到spring容器中**

- @**Import**(AutoConfigurationPackages.Registrar.class):

  spring的底层注解@import,给容器中导入一个组件;

  **AutoConfigurationImportSelector**:导入哪些组件的选择器

  将所有需要导入的组件以犬类名的方式返回:这些组件会被添加到容器中;

  会给容器中导入非常多的自动配置类(xxxAutoConfiguration):就是给容器中导入这个场景需要的所有组件,并配置好这些组件

  有了自动配置类,免去我们手动编写配置注入功能组件等工作;

  **springboot启动的时候从类路径下的META-INF/spring.fatcories中获取EnableAutoConfiguration指定的值,将这些值作为自动配置类导入到容器中,自动配置类就生效,帮我们进行自动配置工作;**

  J2EE的整体整合解决方案和自动配置都在org.springframework.boot.autoconfigure下



### 3. 如何快速创建SpringBoot项目(Spring Initializer)

- File->new->project->Spring Initializer->next->输入包名,项目名->next->选择需要的场景模块->next->finash

- 主程序已经生成好,只需要写自己的业务逻辑

- resources文件夹目录结构

  - static:保存所有的静态资源:js,css,img

  - templates:保存所有的模版页面:(spring boot默认jar包使用嵌入式的tomcat,默认不支持jsp页面),可以使用模版引擎(freemarker,thymeleaf)

  - **application.properties**:Spring Boot应用的配置文件

    ![](C:\Users\Administrator\Desktop\1.png)

### 4. Spring Boot配置文件

1. application.properties
2. application.yml

配置文件的作用:修改SpringBoot自动配置的默认值

- **YAML**

  以前的配置文件大多数使用的都是xml文件,YAML以数据为中心

  YAML:

  ```yml
  server:
    port: 8080
  ```

  XML:

  ```xml
  <server>
  	<port>8080</port>
  </server>
  ```

- **YAML基本语法**

  - K: (**空格**)v:    表示一对键值对

  - 以空格的缩进来控制层级关系:只要是左对齐的一列数据,都是一个层级的

  - 属性和值都是大小写敏感的

  - **值的写法**

    - 字面量

      ```yml
      name: zhangsan
      ```

      字符串默认不用加上单引号或双引号

      双引号:不会转义字符串里面的特殊字符;特殊字符会作为本身想表示的意思

      name:	zhangsan \n 输出:zhangsan 换行

      单引号:会转义特殊字符

      name:	zhangsan \n 输出:zhangsan \n

    - 对象Map

      ```yml
      Person:
      	name: zhangsan
      	age: 18
      	
      Person: {name: zhangsan,age: 18}
      ```

    - 数组List,Set

      ```yml
      Person:
      	- zhangsan
      	- lisi
      	- wangwu
      	
      Person: [zhangsan,lisi,wangwu]
      ```

- **配置文件值注入**

  ```java
  /**
   * @author LiuYunDa
   * @date 2019/10/4 - 14:19
   * 将本类中的所有属性与配置文件中的属性值进行绑定
   * 只有这个组件是容器中的组件,才能使用容器提供的@ConfigurationProperties功能
   */
  
  @Component
  @ConfigurationProperties(prefix = "user")
  public class User {
      private Integer id;
      private String username;
      private Boolean sex;
      private Date birth;
  
      private Map<String,Object> maps;
      private List<Object> list;
      private Dog dog;
  
  ```

  ```yml
  user:
    id: 1
    username: zhangsan
    sex: true
    birth: 2019/10/5
    maps: {k1: v1,k2: v2}
    list: [zhangsan,lisi,wangwu]
    dog: {name: xiaogou,age: 2}
  ```

- @**Value获取值和@ConfigurationProperties获取值比较**

  |                | @ConfigurationProperties | @Value     |
  | -------------- | ------------------------ | ---------- |
  | 功能           | 批量注入配置文件中的属性 | 一个个指定 |
  | 松散绑定       | 支持                     | 不支持     |
  | SpEl           | 不支持                   | 支持       |
  | JSR303数据校验 | 支持                     | 不支持     |
  |                | 支持                     | 不支持     |

  如果说,我们只是在某个业务逻辑中需要获取一下配置文件中的某项值,使用@Value

  如果说,我们专门编写了一个javaBean来和配置文件进行映射,我们就直接使用@ConfigurationProperties

- **配置文件注入值数据校验**

  ```java
  @Component
  @ConfigurationProperties(prefix = "user")
  @Validated
  public class User {
     // @Value("${user.id}")
      @Email
      private Integer id;
      private String username;
      private Boolean sex;
      private Date birth;
  
      private Map<String,Object> maps;
      private List<Object> list;
      private Dog dog;
  ```

- **@PreopertySource和@ImportResource**

  - @PreopertySource:加载指定的配置文件;

    ```java
    @PropertySource(value = ("classpath:User.properties"))
    @Component
    @ConfigurationProperties(prefix = "user")
    //@Validated
    public class User {
       // @Value("${user.id}")
        //@Email
        private Integer id;
        private String username;
        private Boolean sex;
        private Date birth;
    
        private Map<String,Object> maps;
        private List<Object> list;
        private Dog dog;
    ```

  - @ImportResource:导入Spring的配置文件,让配置文件里面的内容生效

    Spring Boot里面没有Spring的配置文件,我们自己编写的配置文件,也不能自动识别,想让Spring的配置文件生效,加载进来,就要用@ImportResource并标注在主配置类上

    ```java
    @ImportResource(locations = "classpath:Beans.xml")
    //导入Spring的配置文件让其生效
    ```

    不来编写Spring的配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="helloService" class="com.qst.service.HelloService"></bean>
    </beans>
    ```

    springboot推荐给容器中添加组件的方式:推荐使用全注解的方式

    1. 配置类=========Spring的配置文件

    2. 使用@Bean给容器中添加组件

       ```java
       /**
        * @author LiuYunDa
        * @date 2019/10/5 - 20:44
        * @Configuration,指明当前类是一个配置类,就是来替代之前的Spring配置文件
        * 在配置文件中使用<bean></bean>标签添加组件
        */
       @Configuration
       public class MyAppConfig {
           //将方法的返回值添加到容器中,容器中这个组件默认的id就是方法名
           @Bean
           public HelloService helloService(){
               System.out.println("配置类@Bean给容器中添加组件了...");
               return new HelloService();
       
           }
       }
       ```

- **配置文件占位符**

  - 随机数

    ```
    ${random.value},${random.int},${random.long},${random.int(10)},${random.int[1024,11002244]}
    ```

    

  - 占位符获取之前配置的值,如果没有可以使用:冒号指定默认值

    ```properties
    user.id=${random.int}
    user.username=张三${random.uuid}
    user.sex=true
    user.birth=2019/5/1
    user.maps.k1=v1
    user.maps.k2=v2
    user.list=a,ab,c
    user.dog.name=${user.hello:you are a}_小狗
    user.dog.age=2
    ```

    

- **Profile**

  是Spring对不同环境提供不同配置功能的支持,可以通过集火,指定参数等方式快速切换环境

  1. 多Profile文件

     我们在主配置文件变写的时候,文件名可以是 application-{profile}.properties/yml

     默认使用application.properties

  2. yml支持多文档块方式

     ```yml
   server:
       port: 8080
   spring:
       profiles:
         active: prod
     ---
     server:
       port: 8081
     spring:
       profiles: dev
     ---
     server:
       port: 8082
     spring:
       profiles: prod
     ```
  
     
  
  3. 激活指定profile
  
     1. application.properties中指定`spring.profiles.active=dev`
     
     2. 命令行:
     
        ![](C:\Users\Administrator\Desktop\2.PNG)
     
        --spring.profiles.active=dev
     
     3. 虚拟机参数
     
        ![](C:\Users\Administrator\Desktop\3.PNG)
     
        -Dspring.profiles.active=dev
  
- **配置文件加载位置**

  1. file:/config/
  2. file:./
  3. classpath:/config/
  4. classpath:/

  以上是按照**优先级从高到低**的顺序,所有位置的文件都会被加载,**高优先级配置内容会覆盖低优先级配置的内容**,会进行**互补配置**

  我们还可以spring.config.location来改变默认的配置文件的位置,**项目打包好以后,我们可以使用命令行参数的形式,启动项目的时候来指定配置文件的新位置,指定配置文件和默认加载的这些配置文件共同形成互补配置**

- 外部配置加载顺序

  略.............

- **自动配置原理**

  1. springboot启动的时候加载主配置类,开启了自动配置功能**@EnableAutoConfiguration**

  2. **@EnableAutoConfiguration**作用:

     - 利用AutoConfigurationImportSelector给容器中导入一些组件

     - 查看导入哪些组件查看selectImports()方法的内容

     - ```java
       //获取候选的配置
       List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
       ```

       - ```java
         SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),      getBeanClassLoader());
         //扫描所有jar包类路径下的META-INF/spring.factories
         //作用是:把扫描到的这些文件的内容包装成properties对象,从properties中获取到EnableAutoConfiguration.class类名对应的值,然后把它们添加在容器中
         ```

         **将类路径下的META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到容器中.** 

         ```properties
         # Auto Configure
         org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
         org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
         org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
         org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
         org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
         org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
         org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
         org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
         org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
         org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
         org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
         org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
         org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
         org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
         org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
         org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
         org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
         org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
         org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
         org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
         org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
         org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
         org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
         org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
         org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
         org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
         org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
         org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
         org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
         org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
         org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
         org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
         org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
         org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
         org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
         org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
         org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
         org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
         org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
         org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
         org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
         org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
         org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
         org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
         org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
         org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
         org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
         org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
         org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
         org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
         org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
         org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
         org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
         org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
         org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
         org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
         org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
         org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
         org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
         org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
         org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
         org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
         org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
         org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
         org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
         org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
         org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
         org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
         org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
         org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
         org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
         org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
         
         ```

         每一个这样的xxxAutoConfiguration类都是容器中的一个组件,都加入到容器中,用他们来做自动配置

  3. **每一个自动配置类进行自动配置功能:**

  4. 以**HttpEncodingAutoConfiguration**为例解释自动配置原理

     ```java
     //表示这是一个配置类
     @Configuration  
     //启用指定类ConfigurationProperties功能:将配置文件中对应的值和HttpProperties绑定起来,并把HttpProperties加入到ioc容器中
     @EnableConfigurationProperties(HttpProperties.class)
      //Spring底层有个Conditional注解,根据不同的条件,如果满足指定的条件,整个配置类里面的配置就会生效.   判断当前应用是否是web应用,如果是当前配置类生效
     @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
     //判断当前项目有没有这个类CharacterEncodingFilter:SpringMVC中进行乱码解决的过滤器
     @ConditionalOnClass(CharacterEncodingFilter.class)
     //判断配置文件中是否存在某个配置spring.http.encoding.enabled.如果不存在也认为这个判断是正确的...即使我们配置文件中不配置spring.http.encoding.enabled=true,也是默认生效的
     @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
     public class HttpEncodingAutoConfiguration {
         //他已经和springboot的配置文件映射了	
         private final HttpProperties.Encoding properties; 
     
         //只有一个有参构造器的情况下,参数的值会从容器中拿
     	public HttpEncodingAutoConfiguration(HttpProperties properties) {
     		this.properties = properties.getEncoding();
     	}
     
         //给容器中添加一个组件,这个组件的某些值需要从properties中获取
     	@Bean  
     	@ConditionalOnMissingBean
     	public CharacterEncodingFilter characterEncodingFilter() {
     		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
     		filter.setEncoding(this.properties.getCharset().name());
     		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
     		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
     		return filter;
     	}
     ```

     根据当前不同的条件判断,决定这个配置类是否生效?

     一但这个配置类生效,这个配置类就会给容器中添加各种组件.这些组件的属性是从对应的properties类中获取的,这些类里面的每一个属性又是和配置文件绑定的

  5. 所有在配置文件中能配置的属性都是在xxxProperties类中封装着,配置文件能配置什么就可以参照某个功能对应的这个属性类

     ```java
     @ConfigurationProperties(prefix = "spring.http") //从配置文件中获取指定的值和bean的属性进行绑定
     public class HttpProperties {
     ```

  6. **精髓:**

     1. **springboot启动会加载大量的自动配置类**
     2. **我们看我们需要的功能有没有在springboot默认写好的自动配置类中;**
     3. **我们再来看这个自动配置类中到底配置了哪些组件,只要们要用的组件有,我们就不需要再来配置了**
     4. **给容器中自动配置类添加组件的时候,会从properties类中获取某些属性,我们就可以在配置文件那种指定这些属性的值**
     5. 我们可以通过启用 debug=true属性,来让控制台打印自动配置报告,这样我们就可以很方便的知道哪些自动配置类生效 

### 5.springboot与日志

1. **日志框架**

   springboot底层是spring框架,spring框架默认使用的是JCL,springboot选用的是SLF4和LogBack

2. **SLF4j使用**

   - 如何在系统中使用SLF4j

     以后开发的时候,日志记录方法的调用,不应该是直接调用日志的实现类,而是调用日志抽象层里面的方法.

     给系统里面导入SLF4j的jar包和logback的实现jar

     ```java
     import org.slf4j.Logger;
     import org.slf4j.LoggerFactory;
     
     public class HelloWorld {
       public static void main(String[] args) {
         Logger logger = LoggerFactory.getLogger(HelloWorld.class);
         logger.info("Hello World");
       }
     }
     ```

     图示

     ![](C:\Users\Administrator\Desktop\concrete-bindings.png)

     每一个日志的实现框架都有自己的配置文件,使用slf4以后,**配置文件还是做成日志实现框架自己本身的配置文件**

3. **遗留问题**

   一个框架可能会用到多种框架,每种框架底层都有不同的日志支持,那么怎么统一使用slf4j如图示:

   ![](C:\Users\Administrator\Desktop\legacy.png)

   1. **将系统中其他的日志框架排除出去**
   2. **用中间包来替换原有的日志框架**
   3. **我们在导入slf4j其他的实现**

4. **springboot日志关系**

   1.  springboot底层也是使用slf4j+logback的方式进行日志记录

   2. SpringBoot也把其他的日志都替换成了slf4j

   3. 中间的替换包 

   4. 如果我们要引入其他框架?一定要把这个框架的默认日志依赖移除掉?

      spring框架用的是commons-logging

      **springboot能自动适配所有的日志,而且底层使用slf4j+logback的方式记录日志,引入其他框架的时候,只需要把这个框架依赖的日志框架排除掉**,

5. **日志的使用**

   ```java
    //日志记录器
       Logger logger = LoggerFactory.getLogger(getClass());
       @Test
       public void contextLoads() {
           //日志的级别:
           //由低到高  trace<debug<info<warn<error
           //可以调整输出的日志级别:日志就只会在这个级别即以后的高级别生效
           //springboot默认给我们使用的是info级别的,没有指定级别的就用springboot默认规定的级别:root级别
           logger.trace("这是trace日志...");
           logger.debug("这是debug日志...");
           logger.info("这是info日志...");
           logger.warn("这是warn日志...");
           logger.error("这是error日志...");
       }
   ```

   springboot修改默认日志的默认配置

   ```properties
   logging.level.com.qst=trace
   #不指定路径在当前项目下生成springboot.log文件
   #可以指定完整的路径,就在路径下生成
   #logging.file=springboot.log
   #在当前磁盘的根路径下创建spring文件夹和里面的log文件夹,使用spring.log作为默认文件
   logging.path=/spring/log
   #在控制台输出的日志的格式
   logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger(50) - %msg%n
   #指定文件中日志输出的格式
   logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger(50) ==== %msg%n
   ```

6. 指定配置

   给类路径下放上每个日志框架的自己的配置文件即可,springboot就不使用它默认的配置了

7. 切换日志框架

   可以按照slf4j的日志适配图,进行相关的切换