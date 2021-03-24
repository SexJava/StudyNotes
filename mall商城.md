



# 谷粒商城

## 1. 环境搭建

### 1.1.  安装Virtual Box

### 1.2. 安装Vagrant

1. 自定义Vagrant Image文件夹

2. 打开cmd窗口切换到Vagrant Image目录

3. 执行命令初始化镜像（采用中科大镜像源）

   ```shell
   vagrant init centos7 https://mirrors.ustc.edu.cn/centos-cloud/centos/7/vagrant/x86_64/images/CentOS-7.box
   ```

4. vagrant up启动镜像

   - 遇到的问题

     - incompatible character encodings: GBK and UTF-8 (Encoding::CompatibilityError)

       - 解决办法：修改virtual box 默认的虚拟电脑位置为自定义路径（不能包含中文）

         ![image-20201224011823629](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201224011823629.png)

5. vagrant ssh连接虚拟机

   - vagrant@127.0.0.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic

     - 解决方法：找到自定义Vagrant Image文件夹下（\Vagrant Images\.vagrant\machines\default\virtualbox）的这个路径下的private_key文件，右键->属性->安全->高级->添加->选择主体->输入对象名（查找位置\当前登录用户名）->检查名称->确定->完全控制->启用继承->选择第二个->确定->确定->

       ![image-20201224012505821](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201224012505821.png)

6. 配置虚拟机网络

   1. 找到自定义Vagrant Image文件夹下的Vagrantfile文件，右键编辑

   2. 修改config.vm.network

      ![image-20201224012818365](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201224012818365.png)

   3. 查看要修改的ip地址

      1. cmd->ipconfig

         ![image-20201224012953098](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201224012953098.png)

      2. 前三位固定，第四位随意

      3. 保存

      4. vagrant reload重启

      5. ip addr 查看当前虚拟机ip

      6. linux 和windows互ping一下试试是否能ping通



### 1.3. 安装Docker

1. 切换root用用户

   ```shell
   su
   后面输入密码
   ```

   ```
   exit;切换回普通用户
   ```

   也可以不切换用户，但是命令前面要加sudo

2. 先卸载

   ```
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine	
   ```

3. 安装依赖包

   ```shell
   sudo yum install -y yum-utils
   ```

4. 设置稳定的存储库

   ```shell
   sudo yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
   ```

5. 安装最新版的docker引擎和容器

   ```shell
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

6. 启动docker

   ```shell
    sudo systemctl start docker
   ```

7. 设置docker开机自启

   ```shell
   sudo systemctl enable docker
   ```

8. 查看docker容器安装的镜像

   ```shell
   sudo docker images 
   ```

9. 配置国内镜像源（阿里云）

   ```shell
   sudo mkdir -p /etc/docker
   ```

   ```shell
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://k9hu0xz3.mirror.aliyuncs.com"]
   }
   EOF
   ```

   ```shell
   sudo systemctl daemon-reload
   ```

   ```shell
   sudo systemctl restart docker
   ```

   

### 1.4. Docker安装MySQL

1. 下载镜像文件

   ```shell
   sudo docker pull mysql:5.7
   ```

2. 查看下载的镜像

   ```shell
   sudo docker images
   ```

3. 创建实例并启动

   ```shell
   sudo docker run -p 3306:3306 --name mysql \
   -v /mydata/mysql/log:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -v /mydata/mysql/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=root \
   -d mysql:5.7
   ```

   参数说明：

   - -p 3306:3306；将容器的3306端口映射到主机的3306端口
   - -v /mydata/mysql/log:/var/log/mysql；将日志文件夹挂载到主机
   - -v /mydata/mysql/data:/var/lib/mysql；将配置文件夹挂载到主机
   - -v /mydata/mysql/conf:/etc/mysql；将配置文件夹挂载到主机
   - -e MYSQL_ROOT_PASSWORD=root ；初始化root用户密码
   - -d mysql:5.7；后台运行指定的mysql5.7这个镜像

4. 查看运行中的容器

   ```shell
   sudo docker ps
   ```

5. 进入mysql容器内部 

   ```shell
   sudo docker exec -it mysql /bin/bash
   ```

6. 退出容器内部

   ```shell
   exit;
   ```

7. 修改mysql的配置文件（字符集）

   ```shell
   vi /mydata/mysql/conf/my.cnf
   ```

   my.cnf内容

   ```
   [client]
   default-character-set=utf8
   
   [mysql]
   default-character-set=utf8
   
   [mysqlId]
   init_connect='SET collation_connection=utf8_unicode_ci'
   init_connect='SET NAMES utf8'
   character-set-server=utf8
   collation-server=utf8_unicode_ci
   skip-character-set-client-handshake
   ship-name-resolve
   ```

8. 重启mysql容器

   ```shell
   sudo docker restart mysql
   ```

9. 自动启动

   ```shell
   docker update mysql --restart=always
   ```

   

### 1.5. Docker安装Redis

1. 获取镜像

   ```shell
   docker pull redis
   ```

2. 创建实例并启动

   ```shell
   mkdir -p /mydata/redis/conf
   touch /mydata/redis/conf/redis.conf
   ```

   

   ```shell
   docker run -p 6379:6379 --name myredis \
   -v /mydata/redis/data:/data \
   -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
   -d redis redis-server /etc/redis/redis.conf
   ```

3. 启动redis控制台

   ```shell
   docker exec -it myredis redis-cli
   ```

4. 配置持久化

   ```shell
   vi redis.conf
   ```

   插入（AOF持久化模式）

   ```shell
   appendonly yes
   ```

5. 重启redis

   ```shell
   docker restart myredis
   ```

6. 配置自动启动

   ```shell
   docker update myredis --restart=always
   ```

   

7. 详细配置信息

   https://raw.githubusercontent.com/redis/redis/4.0/redis.conf



### 1.6. 开发环境

1. maven

   1. 阿里云镜像

      ```xml
      <mirrors>
      	<mirror>
              <id>alimaven</id>
              <mirrorOf>central</mirrorOf>
              <name>aliyun maven</name>
              <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
          </mirror>
      </mirrors>
      ```

   2. maven编译版本

      ```xml
      <profiles>
          <id>JDK-1.8</id>       
          <activation>       
              <activeByDefault>true</activeByDefault>       
              <jdk>1.8</jdk>       
          </activation>       
          <properties>       
              <maven.compiler.source>1.8</maven.compiler.source>       
              <maven.compiler.target>1.8</maven.compiler.target>       
              <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>       
          </properties> 
      </profiles>
      ```

   3. 配置idea使用本地maven

      ![image-20201224234551844](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201224234551844.png)

2. idea插件

   1. lombok
   2. MybatisX

3. 前台环境webstorm

4. 配置git（github）

5. idea搭建项目

6. 拉取github创建的仓库为总项目

7. 创建项目微服务

   1. 商品服务(product)
   2. 仓储服务(ware)
   3. 订单服务 (order)
   4. 优惠券服务(coupon)
   5. 会员服务（member）

8. 服务共同点

   1. 相同的模块：web、openfeign
   2. 每个服务，包名com.lyd.mall.xxx(product)
   3. 模块名，mall-xxx(product)

9. ![image-20201225001309775](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201225001309775.png)

10. 添加mall的总pom

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.lyd.mall</groupId>
        <artifactId>mall</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <name>mall</name>
        <description>商城聚合服务</description>
        <packaging>pom</packaging>
    
        <modules>
           <module>mall-coupon</module>
           <module>mall-order</module>
           <module>mall-member</module>
           <module>mall-product</module>
           <module>mall-ware</module>
        </modules>
    </project>
    
    ```

11. maven中添加刚才添加的pom

    ![image-20201225001845744](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201225001845744.png)
12. 整理各个服务的.gitignore文件



### 1.7. 使用人人开源的renren-fast、renren-fast-vue、renren-generator搭建product微服务

1. 克隆下三个项目

   ```shell
   git clone https://gitee.com/renrenio/renren-fast.git
   ```

   ```shell
   git clone https://gitee.com/renrenio/renren-fast-vue.git
   ```

   ```shell
   git clone https://gitee.com/renrenio/renren-generator.git
   ```

2. 将三个文件中的.git文件删掉

3. 将renren-fast/renren-generator，放入mall项目目录

4. 将其添加到pom中

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.lyd.mall</groupId>
       <artifactId>mall</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <name>mall</name>
       <description>商城聚合服务</description>
       <packaging>pom</packaging>
   
       <modules>
           <module>mall-coupon</module>
           <module>mall-order</module>
           <module>mall-member</module>
           <module>mall-product</module>
           <module>mall-ware</module>
           <module>renren-fast</module>
           <module>renren-generator</module>
           <module>mall-common</module>
       </modules>
   </project>
   ```

5. 配置renren-fast

   1. 新建mall_admin数据库

      ![image-20201228224717044](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201228224717044.png)

   2. 执行这个sql创建表结构

   3. 修改application-dev.yml中的数据库连接地址改为自己的

6. 配置renren-fast-vue

   1. 前提是安装node环境（略）
   2. 删掉.git文件
   3. 用webstorm打开
   4. 打开Terminal
   5. 运行`npm install`
   6. 运行`npm run dev`

7. 至此后台管理项目搭建成功

8. 采用代码生成工具生成微服务增删改查代码（配置renren-generator生成product微服务模块代码）

   1. 配置application.yml，数据连接地址为微服务模块的数据库地址

   2. 配置generator.properties

      ![image-20201228225542492](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201228225542492.png)

   3. 启动项目

   4. 全选点击生成代码

      ![image-20201228225631894](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201228225631894.png)

   5. 将生成的代码复制到微服务模块中

9. 处理生成的代码的报错情况

   1. 新建公共module
   2. 将缺少的还有报错的代码都复制到这个模块
   3. 在微服务模块的pom中引用该module
   4. 即可

10. 微服务模块整合MyBatis-Plus

    1. 官方文档

       > https://baomidou.com/

    2. 导入依赖

       ```xml
       <!--mybatis-plus-->
       <dependency>
           <groupId>com.baomidou</groupId>
           <artifactId>mybatis-plus-boot-starter</artifactId>
           <version>3.4.1</version>
       </dependency>
       ```

    3. 配置

       1. 数据源

          1. 导入数据库的驱动

             ```xml
             <!--mysql驱动-->
             <dependency>
                 <groupId>mysql</groupId>
                 <artifactId>mysql-connector-java</artifactId>
                 <version>8.0.19</version>
             </dependency>
             ```

          2. 配置数据源

             ```yml
             spring:
               datasource:
                 username: root
                 password: root
                 url: jdbc:mysql://192.168.56.10:3306/mall_pms?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
                 driver-class-name: com.mysql.cj.jdbc.Driver
             ```

       2. 配置Mybatis-Plus

          1. 主启动类加上`@MapperScan("com.lyd.mall.product.dao")`注解

          2. 配置Mybatis-puls，sql映射文件的位置

             ```shell
             mybatis-plus:
               mapper-locations: classpath:/mapper/**/*.xml
             ```

          3. 配置自增主键

             ```shell
             mybatis-plus:
               mapper-locations: classpath*:/mapper/**/*.xml
               global-config:
                 db-config:
                   id-type: auto
             ```

       3. crud测试

       4. 生成所有微服务代码（略）

## 2. 微服务

### 2.1. 微服务组件搭配方案

1. SpringCloud Alibaba-Nacos：注册中心（服务发现/注册）
2. SpringCloud Alibaba-Nacos：配置中心（动态配置管理）
3. SpringCloud-Ribbon：负载均衡
4. SpringCloud-Feign：声明式HTTP客户端（远程调用服务）
5. SpringCloud Alibaba-Sentinel：服务容错（限流、降级、熔断）
6. SpringCloud-GateWay：API网关（webflux编程模式 ）
7. SpringCloud-Sleuth：调用链路监控
8. SpringCloud Alibaba-Seata：原Fescar，即分布式事务解决方案

### 2.2. 版本选择

- SpringCloud 1.5x版本是用于 SpringBoot 1.5x

- SpringCloud 2.0x版本是用于 SpringBoot 2.0x

- SpringCloud 2.1x版本是用于 SpringBoot 2.1x

  ![image-20201229233136277](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201229233136277.png)

### 2.3. 项目中的依赖

在common项目中引入，进行统一管理

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2.4. SpringCloud Alibaba-Nacos（作为注册中心）

1. 修改pom引入 Nacos Discovery Starter（common项目中引入）。

   ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
   ```

2. 配置文件中配置nacos server地址

   ```properties
    spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   ```

3. 使用 @EnableDiscoveryClient 注解开启服务注册与发现功能（主启动类）

4. 下载nacos-server

5. 启动nacos-server

### 2.5. SpringCloud-Feign（声明式远程调用）

1. 引入pom依赖

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 编写一个接口告诉SpringCloud这个接口需要调用远程服务

   ```java
   @FeignClient("mall-coupon")
   public interface CouponFeignService {
       @RequestMapping("/coupon/coupon/member/list")
       public R memberCoupons();
   }
   
   ```

3. 开启远程调用功能

   ```java
   @EnableFeignClients(basePackages = "com.lyd.mall.member.feign")
   ```

4. 出现问题，启动member项目报错说未引用什么什么 loadBalancing

   降低springboot版本从2.4.x降到2.2.x，对应的springcloud版本从2020.0.0降到Hoxton.SR9

   原因，是因为springcloud2020版本重大改革，替换了之前用的部分netflix组件，在网上找了张图片

   ![image-20201229232630545](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201229232630545.png)

### 2.6. SpringCloud Alibaba-Nacos（作为配置中心）

1. 首先，修改 pom.xml 文件，引入 Nacos Config Starter。

   ```xml
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
   ```

2. 在应用的 /src/main/resources/bootstrap.properties 配置文件中配置 Nacos Config 元数据

   bootstrap.properties会优先于application.properties加载

   ```properties
   spring.application.name=mall-coupon
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   ```

3. 完成上述两步后，应用会从 Nacos Config 中获取相应的配置，并添加在 Spring Environment 的 PropertySources 中。这里我们使用 @Value 注解来将对应的配置注入到 SampleController 的 userName 和 age 字段，并添加 @RefreshScope 打开动态刷新功能

   ```java
   @RefreshScope
    class SampleController {
   
    	@Value("${user.name}")
    	String userName;
   
    	@Value("${user.age}")
    	int age;
    }
   ```

4. Nacos 新建配置

   ![image-20201229235027075](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201229235027075.png)

   ![image-20201229235050136](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201229235050136.png)

5. 注意

   如果配置张总新和当前的配置文件中都配置了相同的项，优先使用配置中心的配置

6. 细节

   - 分类配置：NameSpace + Group + Data ID

     ![image-20201202231856124](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201202231856124.png)

     ![image-20201202233917848](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201202233917848.png)

   - 将application.properties里面的配置都部署在nacos

     - 例如将配置分类

       - 数据源配置
       - Mybatis配置
       - 其他配置

     - 在nacos新建三个配置文件

       ![image-20201230002855693](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20201230002855693.png)

     - 对应的配置信息就是application.properties中配置的信息

     - bootstrap.properties配置多配置集

       ```properties
       spring.application.name=mall-coupon
       spring.cloud.nacos.config.server-addr=127.0.0.1:8848
       spring.cloud.nacos.config.namespace=b20043d8-c775-4cfd-b866-24f9d8803720
       spring.cloud.nacos.config.group=dev
       spring.cloud.nacos.config.extension-configs[0].data-id=datasource.yml
       spring.cloud.nacos.config.extension-configs[0].group=dev
       spring.cloud.nacos.config.extension-configs[0].refresh=true
       
       spring.cloud.nacos.config.extension-configs[1].data-id=mybatis.yml
       spring.cloud.nacos.config.extension-configs[1].group=dev
       spring.cloud.nacos.config.extension-configs[1].refresh=true
       
       spring.cloud.nacos.config.extension-configs[2].data-id=other.yml
       spring.cloud.nacos.config.extension-configs[2].group=dev
       spring.cloud.nacos.config.extension-configs[2].refresh=true
       ```

       

   - Nacos集群和持久化配置

     - Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL数据库的存储

     - 三种部署模式

       - 单机模式-用于测试和单机试用
       - 集群模式-用于生产环境，确保高可用
       - 多集群模式-用于多数据中心场景

     - Nacos默认自带的是嵌入式数据库derby

     - derby到mysql的切换配置步骤

       - nacos-server-1.1.4\nacos\conf目录下找到sql脚本

         - nacos-mysql.sql
         - 执行脚本

       - nacos-server-1.1.4\nacos\conf目录下找到application.properties

         ```properties
         spring.datasource.platform=mysql
         
         db.num=1
         db.url.0=jdbc:mysql://127.0.0.1:3306/数据库名?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
         db.user=root
         db.password=123456
         ```

         

### 2.7. SpringCloud-Gateway（网关）

网关作为流量的入口，常用功能包括路由转发，权限校验，限流控制等。

1. 开启服务注册发现，主启动类添加`@EnableDiscoveryClient`注解

2. 配置

   ```yml
   spring:
     cloud:
       gateway:
         routes:
           - id: mall-gateway-baidu
             uri: https://www.baidu.com
             predicates:
               - Query=url,baidu
           - id: mall-gateway-qq
             uri: https://www.qq.com
             predicates:
               - Query=url,qq
   ```



### 2.8. 前端开发基础

#### 2.8.1 ES6

1. let声明变量

   ```js
   
   // var 声明的变量往往会越域
   // let 声明的变量有严格局部作用域
   // {
   //     var a = 1;
   //     let b = 2;
   // }
   // console.log(a); // 1
   // console.log(b); // Uncaught ReferenceError: b is not defined
   // var 可以声明多次
   // let 只能声明一次
   // var m = 1;
   // var m = 2;
   // let n = 3;
   // // let n = 4;
   // console.log(m); // 2
   // console.log(n); // Uncaught SyntaxError: Identifier 'n' has already been declared
   // var 会变量提升
   // let 不存在变量提升
   console.log(x); // undefined
   var x = 10;
   console.log(y); // Uncaught ReferenceError: Cannot access 'y' before initialization
   let y = 20;
   ```

2. const声明变量（只读变量）

   ```js
   // 声明变量之后不允许改变
   // 一旦声明必须初始化，否则会报错
   const a= 1;
   a = 2; // Uncaught TypeError: Assignment to constant variable.
   ```

3. 解构表达式

   1. 数组解构

      ```js
      let arr = [1,2,3];
      // let a = arr[0];
      // let b = arr[1];
      // let c = arr[2];
      let [a,b,c] = arr;
      console.log(a);
      console.log(b);
      console.log(c);
      ```

      

   2. 对象解构

      ```js
      const person = {
      name: "zhangsan",
      age: 20,
      language: ['java', 'js', 'css']
      }
      // const name = person.name;
      // const age = person.age;
      // const language = person.language;
      const {name:firstname, age, language} = person
      console.log(firstname, age, language);
      ```

4. 字符串扩展

   1. 新增API

      ```js
      let str = "hello vue";
      console.log(str.startsWith("hello")); // true
      console.log(str.endsWith("vue")); // true
      console.log(str.includes("e")); // true
      console.log(str.includes("hello")); // true
      ```

   2. 字符串模版

      ```js
      // 字符串模版
      let ss = `<div>
      <span>hello world</span>
      </div>`;
      console.log(ss);
      // 字符串插入变量和表达式。变量卸载${}中，${}中可以放入JavaScript表达式
      let name = "zhangsan";
      let age  = 10;
      function fun() {
          return "这是一个函数";
      }
      let info = `我是${name},今年${age+10}了,我想说:${fun()}`;
      console.log(info);
      ```

5. 函数优化

   1. 函数参数默认值

      ```js
          // es6之前，我们无法给函数参数设置默认值，只能采用变通写法
          function add(a,b) {
              // 判断b是否为空，为空给默认值1
              b = b || 1;
              return a + b;
          }
          function add2(a,b =1){
              return a + b;
          }
          console.log(add(10));
          console.log(add2(10));
      ```

      

   2. 不定参数

      ```js
          // 不定参数
          function fun(...values) {
              console.log(values.length)
          }
          fun(1,2) // 2
          fun(1,2,3) // 3
      ```

      

   3. 箭头函数

      ```js
          // 箭头函数
          // 以前声明一个方法
          var print = function (object) {
              console.log(object)
          }
          var print = obj => console.log(obj);
          print("hello")
      
          var sum2 = (a,b) => a+b;
          console.log(sum2(11,12))
          var sum3 = (a,b) => {
              c = a+b;
              return c;
          };
          console.log(sum3(10,10));
      ```

      

   4. 实战：箭头函数结合解构表达式

      ```js
          const person = {
              name : "zhangsan",
              age : 10,
              language: ['java','js']
          }
          // es6以前
          function hello(person) {
              console.log("hello,"+person.name)
          }
          // 箭头+解构
          var hello2 = ({name}) => console.log("hello,"+name);
          hello2(person)
      ```

6. 对象优化

   1. 新增API

      ```js
          const person = {
              name : "zhangsan",
              age : 10,
              language: ['java','js']
          }
      
          console.log(Object.keys(person)) // 0: "name" 1: "age" 2: "language"
          console.log(Object.values(person)) // 0: "zhangsan" 1: 10 2: (2) ["java", "js"]
          console.log(Object.entries(person)) // 0: (2) ["name", "zhangsan"] 1: (2) ["age", 10] 2: (2) ["language", Array(2)]
      
          const target = {a:1};
          const source1 = {b:2};
          const source2 = {c:3};
      
          // { a:1,b:2,c:3}
          Object.assign(target,source1,source2)
          console.log(target)
      ```

   2. 声明对象简写

      ```js
          // 声明对象简写
          const age = 23;
          const name = "lisi";
          // 以前
          const person1 = {age:age,name:name}
          // es6属性名如果和属性值的变量名相同，可简写
          const person2 = {age,name}
          console.log(person1)
          console.log(person2)
      ```

      

   3. 对象函数属性的简写

      ```js
          // 对象的函数属性简写
          let person3 = {
              name : "wangwu",
              // 以前
              eat:function (food) {
                  console.log(this.name+"吃"+food);
              },
              // es6 之后
              // 箭头函数this不能使用，对象.属性
              eat2:food=>console.log(person3.name+"吃"+food),
              eat3(food){
                  console.log(this.name+"吃"+food);
              }
      
      
          }
          person3.eat("xiang");
          person3.eat2("xiang");
          person3.eat3("xiang");
      ```

      

   4. 对象的扩展运算符

      ```js
          // 扩展运算符
          // 拷贝对象（深拷贝）
          let person4 = {name:'赵六',age:90};
          let someone = {...person4};
          console.log(someone)
          // 合并对象
          let age1 = {age :0};
          let name1 = {name:'张三'}
          // 如果量对象的字段名重复，后面对象的字段值会覆盖前面对象的字段值
          let person5 = {...age1,...name1}
          console.log(person5)
      ```

7. map和reduce

   ```js
       // 数组中新增了map和reduce方法
       // map();接收一个函数，将原数组中的所有元素用这个函数处理后放入新数组返回
       let arr = ['1','20','-5','3'];
       // arr = arr.map((item)=>{
       //     return item*2;
       // })
   
       arr = arr.map(item=>item*2)
       console.log(arr)
       // reduce();为数组中的每个元素一次执行回调函数，不包括数组中被删除或从未被赋值的元素
       /**
        * reduce(callback,[initialValue])
        * callback的参数
        * 1.previousValue 上次调用回调返回的值，或者是提供的初始值(initialValue)
        * 2.currentValue 数组中当前被处理的元素
        * 3。index 当前元素在数组中的索引
        * 4.array 调用reduce的数组
        */
       let result = arr.reduce((a,b)=>{
           console.log("上一次处理后："+a);
           console.log("当前正在处理："+b);
           return a + b;
       },100);
       console.log(result)
   ```

8. Promise

   ```js
       // 1.查出当前用户信息
       // 2.根据当前用户的id查出他的课程
       // 3.根据当前课程id查出份数
       // $.ajax({
       //     url: "mock/user.json",
       //     success(data){
       //         console.log("查询用户：",data)
       //         $.ajax({
       //             url: `mock/user_corse_${data.id}.json`,
       //             success(data) {
       //                 console.log("查询到课程",data)
       //                 $.ajax({
       //                     url:`mock/corse_score_${data.id}.json`,
       //                     success(data){
       //                         console.log("查询到的分数",data)
       //                     }
       //                 })
       //             }
       //         })
       //     }
       // })
   
       // 1.Promise可以封装异步操作
       // let p = new Promise((resolve, reject) => {
       //     // 1.异步操作
       //     $.ajax({
       //         url: "mock/user.json",
       //         success(data) {
       //             console.log("查询用户：",data)
       //             resolve(data)
       //         },
       //         error(error){
       //             reject(error)
       //         }
       //     })
       // })
       // p.then((object) => {
       //     return new Promise((resolve1, reject1) => {
       //         $.ajax({
       //             url: `mock/user_corse_${object.id}.json`,
       //             success(data) {
       //                 console.log("查询到课程", data)
       //                 resolve1(data)
       //             },
       //             error(error) {
       //                 reject1(error)
       //             }
       //         })
       //     })
       // }).then((object) => {
       //     $.ajax({
       //         url: `mock/corse_score_${object.id}.json`,
       //         success(data) {
       //             console.log("查询到的分数", data)
       //         }
       //     })
       // })
   
       // 2. promise 优化
       function get(url,data) {
           return new Promise((resolve, reject) => {
               $.ajax({
                   url:url,
                   data:data,
                   success(data){
                       resolve(data)
                   },
                   error(error){
                       reject(error)
                   }
               })
           })
       }
   
       get("mock/user.json").then((data) => {
           console.log("用户查询成功", data)
           return get(`mock/user_corse_${data.id}.json`)
       }).then((data) => {
           console.log("查询到课程", data)
           return get(`mock/corse_score_${data.id}.json`)
       }).then(data => {
           console.log("查询到的分数", data)
       }).catch(err => {
           console.log("异常：" + err)
       })
   ```

   

9. 模块化

   1. 什么是模块化

      就是把代码进行拆分，方便重复利用。类似java的导包

   2. export

      用于规定模块对外接口

   3. import

      用于导入其他模块提供的功能

#### 2.8.2 Vue

1. MVVM思想
   - M：Model，模型，包括数据和一些基本操作
   - V：View，视图，页面渲染结果
   - VM：View-Model，模型与视图之间的双向操作





## 3. 商品服务

### 3.1. 跨域

指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成 的，是浏览器对JavaScript施加的安全限制

### 3.2. 同源策略

指的是协议，域名，端口都要相同，其中有一个不同都会产生跨域

### 3.3. 跨域流程

![image-20210112001839188](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210112001839188.png)

### 3.4.解决跨域

1. 使用nginx部署为同一域 

   ![image-20210112002106640](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210112002106640.png)

2. 配置当次请求允许跨域

   ![image-20210112002308229](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210112002308229.png)

### 3.5. MyBatis-Plus逻辑删除

1. 配置全局的逻辑删除规则

2. 给Bean加上逻辑删除注解@TableLogic

   ![image-20210112221420615](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210112221420615.png)

## 4.基础篇总结

![image-20210304233729217](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210304233729217.png)

## 5.高级篇

### 5.1.Elasticsearch

### 5.2.商品上架

ES在内存中，所以由于mysql。es也支持集群，数据库分片存储

- 索引方案

  ```
  {
      skuId:1
      spuId:11
      skyTitile:华为xx
      price:999
      saleCount:99
      attr:[
          {尺寸:5},
          {CPU:高通945},
          {分辨率:全高清}
  	]
  }
  缺点：如果每个sku都存储规格参数(如尺寸)，会有冗余存储，因为每个spu对应的sku的规格参数都一
  ```

  ```
  sku索引
  {
      spuId:1
      skuId:11
  }
  attr索引
  {
      skuId:11
      attr:[
          {尺寸:5},
          {CPU:高通945},
          {分辨率:全高清}
  	]
  }
  先找到4000个符合要求的spu，再根据4000个spu查询对应的属性，封装了4000个id，long 8B*4000=32000B=32KB
  1K个人检索，就是32MB
  
  
  结论：如果将规格参数单独建立索引，会出现检索时出现大量数据传输的问题，会引起网络网络
  
  ```

  选用第一个方案，空间换时间

  **建立Product索引**

  ```json
  PUT product
  {
      "mappings":{
          "properties": {
              "skuId":{ "type": "long" },
              "spuId":{ "type": "keyword" },  # 不可分词
              "skuTitle": {
                  "type": "text",
                  "analyzer": "ik_smart"  # 中文分词器
              },
              "skuPrice": { "type": "keyword" },
              "skuImg"  : { "type": "keyword" },
              "saleCount":{ "type":"long" },
              "hasStock": { "type": "boolean" },
              "hotScore": { "type": "long"  },
              "brandId":  { "type": "long" },
              "catalogId": { "type": "long"  },
              "brandName": {"type": "keyword"},
              "brandImg":{
                  "type": "keyword",
                  "index": false,  # 不可被检索，不生成index
                  "doc_values": false # 不可被聚合
              },
              "catalogName": {"type": "keyword" },
              "attrs": {
                  "type": "nested",
                  "properties": {
                      "attrId": {"type": "long"  },
                      "attrName": {
                          "type": "keyword",
                          "index": false,
                          "doc_values": false
                      },
                      "attrValue": {"type": "keyword" }
                  }
              }
          }
      }
  }
  ```

  - “type”: “keyword” 保持数据精度问题，可以检索，但不分词
  - “index”:false 代表不可被检索
  - “doc_values”: false 不可被聚合，es就不会维护一些聚合的信息

  冗余存储的字段：不用来检索，也不用来分析，节省空间。

  库存是bool。

  检索品牌id，但是不检索品牌名字、图片

  用skuTitle检索

- ### nested嵌入式对象

  属性是"type": “nested”,因为是内部的属性进行检索

  数组类型的对象会被扁平化处理（对象的每个属性会分别存储到一起）

  ```json
  user.name=["aaa","bbb"]
  user.addr=["ccc","ddd"]
  
  这种存储方式，可能会发生如下错误：
  错误检索到{aaa,ddd}，这个组合是不存在的
  
  ```

  数组的扁平化处理会使检索能检索到本身不存在的，为了解决这个问题，就采用了嵌入式属性，数组里是对象时用嵌入式属性（不是对象无需用嵌入式属性）

  nested阅读：https://blog.csdn.net/weixin_40341116/article/details/80778599

  使用聚合：https://blog.csdn.net/kabike/article/details/101460578

### 5.3.Nginx配置反向代理

> http://nginx.org/en/

Hosts文件

```
192.168.56.10 mall.com
```

Nginx配置文件

![image-20210324210936104](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324210936104.png)

默认配置

```text

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

Server块

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

- 不走网关

  mall.conf

  ```
  server {
      listen       80;
      server_name  mall.com;
  
      #charset koi8-r;
      #access_log  /var/log/nginx/log/host.access.log  main;
  
      location / {
  		proxy_pass http://192.168.56.1:10000;
      }
  ……………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………………
  ```

  

- 走网关

  ![image-20210324213129664](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324213129664.png)

nginx.conf

```
………………………………………………


http {
    ……………………………………

    #gzip  on;
    
    upstream mall{
	server 192.168.56.1:88;
    }
    include /etc/nginx/conf.d/*.conf;
}

```

mall.conf

```
server {
    listen       80;
    server_name  mall.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
		proxy_pass http://mall;
    }

   ………………………………………………………………………………………………………………………………………………
}

```

配置gateway

> https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

配置到所有路由的最下面（粗粒度匹配的放在下面）

```
spring:
  cloud:
    gateway:
      routes:
      - id: mall_host_route
        uri: lb://mall-product
        predicates:
        - Host=**.mall.com
```

**Nginx代理给网关的时候，会丢失请求的hots信息**

mall.conf

```
server {
    listen       80;
    server_name  mall.com;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
		proxy_set_header Host $host;
		proxy_pass http://mall;

    }

   ………………………………………………………………………………………………………………………………………………
}

```

