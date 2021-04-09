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

### 5.4.压力测试

> 压力测试考察当前软硬件环境下系统所能承受的最大负荷并帮助找出系统瓶颈所在。压测都是为了系统在线上的处理能力和稳定性维持在一个标准范围内，做到心中有数
>
> 使用压力测试，我们有希望找到很多种用其他测试方法更难发现的错误。有两种错误类型是：内存泄漏，并发与同步。
>
> 有效的压力测试系统将应用一下这些关键条件：重复，并发，量级，随机变化。

**性能指标**

- 响应时间(Response Time: RT)

> 响应时间指用户从客户端发起一个请求开始,到客户端接收到从服务器端返回的响应结束,整个过程所耗费的时间。

- HPS (Hits Per Second) :每秒点击次数,单位是次秒。
- TPS (Transaction per Second);,系統每秒处理交易数,单位是笔秒。
- OPS (Query per Second) :系统每秒处理查询次数,单位是次秒。

> 对于互联网业务中,如果某些业务有且仅有一个请求连接,那么TPS-QPS-HPS,一般情况下用TPS来衡量整个业务流程,用QPS来衡量接口查询次数,用HPS来表示对服务器单击请求。

- 无论TPS. QPS. HPS,此指标是衡量系统处理能力非常重要的指标,越大越好,根据经验,一般情况下:

> 金融行业: 1000TPS-50000TPs,不包括互联网化的活动
> 保险行业: 100TPS-100000Tps. 包括互联网化的活动
> 制造行业: 10TPS-5000TPS
> 互联网电子商务: 1000OTPS-1000000TPS
> 互联网中型网站: 1000TPS-50000TPS
> 互联网小型网站: 500TPS~1000TPS

- 最大响应时间(Max Resonse Time,指用户发出请求或者指令到系统做出反应(响应)的最大时间。
- 最少响应时间(Mininum RespanseTime)指用户发出请求或者指令到系统做出反应(响应)的最少时间。
- 90%响应时间(90% Response Time) 是指所有用户的响应时间进行排序,第90%的响应时间。
- 从外部看,性能测试主要关注如下三个指标

> 吞吐量;每秒钟系统能够处理的请求数、任务数.
> 响应时间:服务处理一个请求或一个任务的耗时。
> 错误率:一批请求中结果出错的请求所占比例。

1. 修改中文

   ![image-20210324230726265](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324230726265.png)

2. 添加线程组模拟用户

   ![image-20210324230805182](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324230805182.png)

3. 模拟200个用户1s内启动，循环100次，即20000个请求

   ![image-20210324230843372](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324230843372.png)

4. 添加http请求

   ![image-20210324230908608](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324230908608.png)

5. 请求接口

   ![image-20210324230948500](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324230948500.png)

6. 添加查看结果树，汇总报告

   ![image-20210324231038054](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210324231038054.png)

**jconsole与jvisualvm**

> Jdk的两个小工具console visulvm (升级版的consoke),通过命令行启动,可监控本地和远程应用.远程应用需要配置
>
> https://zhuanlan.zhihu.com/p/75799243

- jconsole

  - 命令行 jconsole即可

- jvisualvm

  ![image-20210325204024334](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210325204024334.png)

  运行：正在运行的

  休眠：sleep

  等待：wait

  驻留：线程池里面的空闲线程

  监视：阻塞的线程，正在等待锁

### 5.5.Nginx动静分离

- 将所有项目的静态资源都应该放在nginx里面

- 规则：/static/**的所有请求都由nginx直接返回

  mall.conf

```
location /static/ {
    root   /usr/share/nginx/html;
} 
```

### 5.5.缓存与分布式锁

1. 缓存

   1. 缓存使用

      为了系统性能的提升,我们一般都会将部分数据放入缓存中,加速访问。而db承担数据落盘工作。

      **哪些数据适合放进缓存？**

      - 即时性、数据一致性要求不高的

      - 访问量大且更新频率不高的数据(读多,写少)
        举例:电商类应用,商品分类,商品列表等适合缓存并加一个失效时间(根据数据更新频率来定),后台如果发布一个商品,买家需要5分钟才能看到新的商品一般还是可以接受的。

        ![image-20210330212813222](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210330212813222.png)

   2. 整合redis作为缓存

      - pom

        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
        ```

      - application.yml

        ```yml
        spring:
          redis:
            host: 192.168.56.10
            port: 6379
        ```

      - 

2. 缓存失效问题

   1. 缓存穿透

      指**查询一个一定不存在的数据**，由于缓存是不命中，将去查询数据库，但是数据库也没有此记录，我们没有将这次查询的null写入缓存，这将导致这个不存在的数据每次请求都要去存储层去查询，失去了缓存的意义

      - 风险：利用不存在的数据进行攻击，数据库瞬时压力增大，最终导致崩溃
      - 解决：null结果缓存，并加入短暂过期时间

   2. 缓存雪崩

      指在我们设置缓存时**key采用了相同的过期时间**，导致缓存在某一时刻同时失效，请求全部转发到数据库，数据库瞬时压力过大导致雪崩

      - 解决：原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件

   3. 缓存击穿

      对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种**非常热点的数据**

      如果这个**key在大量请求同时进来前正好失效**，那么所有对这个key的数据查询到落到数据库，我们成为缓存击穿

      - 解决：加锁，大量并发只让一个人去查，其他人等待，查到以后释放锁，其他人获得锁，先查缓存，就会有数据，不用去查数据库

3. 分布式锁

   1. 分布式锁基本原理

      ![image-20210331202321720](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210331202321720.png)

      我们可以同时去一个地方“占坑”，如果站到，就执行逻辑。否则就必须等待，直到释放锁。“占坑”可以去redis，也可以去数据库，可以去任何大家都能访问的地方。

      等待可以自旋的方式

   2. 流程

      ![image-20210331212351743](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210331212351743.png)

      问题：setnx占好了位，业务代码异常或者程序在页面过程中宕机。没有执行删除锁逻辑，这就造成了**死锁**

      解决：设置锁的自动过期，即使没有删除，也会自动删除

      问题：删除锁直接删除么？由于业务时间很长，锁自己过期了，我们直接删除，有可能把别人正在持有的锁删除了

      解决：占锁的时候，值指定为uuid，每个人匹配是自己的锁才删除

      问题：如果正好判断是当前值，正要删除锁的时候，锁已经过期了，别人已经设置了新的值。那么我们删除的是别人的锁

      解决：删除锁必须保证原子性。使用redis+Lua脚本完成

      ```java
          public Map<String, List<CateLog2Vo>> getCatalogJsonFromDbWithRedisLock() {
              // 1.占分布式锁。去redis占坑
              // 设置过期时间，必须和加锁是同步的，原子的
              String token = UUID.randomUUID().toString();
              Boolean lock = stringRedisTemplate.opsForValue().setIfAbsent("lock", token,300,TimeUnit.SECONDS);
              Map<String, List<CateLog2Vo>> dataFromDb;
              if (lock){
                  // 加锁成功...执行业务
                  try {
                      dataFromDb = getDataFromDb();
                  }finally {
                      // 获取值对比+对比成功删除 原子操作
                      String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
                      // 删除锁
                      Long lock1 = stringRedisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class)
                              , Arrays.asList("lock")
                              , token);
                  }
      
                  return dataFromDb;
              }else {
                  // 加锁失败...重试。synchronized()
                  // 自旋
                  // 休眠100ms
                  try {
                      Thread.sleep(100);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  return getCatalogJsonFromDbWithRedisLock();
      
              }
          }
      ```

   3. **Redisson**

      > https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95

      - 概述：

        Redisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务。其中包括(`BitSet`, `Set`, `Multimap`, `SortedSet`, `Map`, `List`, `Queue`, `BlockingQueue`, `Deque`, `BlockingDeque`, `Semaphore`, `Lock`, `AtomicLong`, `CountDownLatch`, `Publish / Subscribe`, `Bloom filter`, `Remote service`, `Spring cache`, `Executor service`, `Live Object service`, `Scheduler service`) Redisson提供了使用Redis的最简单和最便捷的方法。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上。

      - 整合redisson作为分布式锁等功能的框架

        - pom.xml

          ```xml
          <!--redisson-->
          <dependency>
              <groupId>org.redisson</groupId>
              <artifactId>redisson</artifactId>
              <version>3.12.0</version>
          </dependency>
          ```

        - 程序化配置

          ```java
          @Configuration
          public class MyRedissonConfig {
          
              /**
               * @Description: 所有对redisson的使用都是通过RedissonClient对象
               * @Param: []
               * @return: org.redisson.api.RedissonClient
               * @Author: Liuyunda
               * @Date: 2021/4/1
               */
              @Bean(destroyMethod="shutdown")
              public RedissonClient redisson() throws IOException {
                  // 创建配置
                  Config config = new Config();
                  // 安全链接用rediss://
                  config.useSingleServer().setAddress("redis://192.168.56.10:6379");
                  // 根据config创建出redissonClient实例
                  RedissonClient redissonClient = Redisson.create(config);
                  return redissonClient;
          
              }
          }
          ```

        - **可重入锁（Reentrant Lock）**

          > 基于Redis的Redisson分布式可重入锁[`RLock`](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLock.html) Java对象实现了`java.util.concurrent.locks.Lock`接口。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockRx.html)的接口。

          ```java
              @ResponseBody
              @GetMapping("/hello")
              public String hello(){
                  // 获取锁，只要锁的名字一样，就是同一把锁
                  RLock lock = redisson.getLock("my-lock");
                  // 加锁
                  // 阻塞式等待（看门狗自动续期）
                  // lock.lock();
                  /**
                   * 10秒自动解锁，自动解锁时间一定要大于业务的执行时间
                   * 如果传递了锁的超时时间，就发送给redis执行脚本，进行占锁，默认超时时间就是我们指定的时间
                   * 如果我们没指定锁的超时时间，就使用LockWatchdogTimeout（看门狗）的默认时间，
                   *    只要占锁成功，就会启动一个定时任务（重新给锁设置过期时间，新的过期时间就是看门狗的默认时间）
                   *    每隔（internalLockleaseTime(看门狗时间)/3）时间进行一次续期操作
                   */
                  // lock.lock(10, TimeUnit.SECONDS);
                  // 最佳实战(省掉了整个续期操作，手动解锁)
                  lock.lock(30, TimeUnit.SECONDS);
                  try {
                      System.out.println("加锁成功，执行业务。。。。"+Thread.currentThread().getId());
                      Thread.sleep(30000);
                  }catch (Exception e) {
          
          
                  } finally {
                      // 解锁
                      System.out.println("释放锁。。。"+Thread.currentThread().getId());
                      lock.unlock();
                  }
                  return "hello";
              }
          ```

          ```java
          // 加锁以后10秒钟自动解锁
          // 无需调用unlock方法手动解锁
          lock.lock(10, TimeUnit.SECONDS);
          
          // 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
          boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
          if (res) {
             try {
               ...
             } finally {
                 lock.unlock();
             }
          }
          ```

          > 大家都知道，如果负责储存这个分布式锁的Redisson节点宕机以后，而且这个锁正好处于锁住的状态时，这个锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-配置方法#lockwatchdogtimeout监控锁的看门狗超时单位毫秒)来另行指定。
          >
          > 另外Redisson还通过加锁的方法提供了`leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

        - **公平锁（Fair Lock）**

          > 基于Redis的Redisson分布式可重入公平锁也是实现了`java.util.concurrent.locks.Lock`接口的一种`RLock`对象。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RLockRx.html)的接口。它保证了当多个Redisson客户端线程同时请求加锁时，优先分配给先发出请求的线程。所有请求线程会在一个队列中排队，当某个线程出现宕机时，Redisson会等待5秒后继续下一个线程，也就是说如果前面有5个线程都处于等待状态，那么后面的线程会等待至少25秒。

          ```java
          RLock fairLock = redisson.getFairLock("anyLock");
          // 最常见的使用方法
          fairLock.lock();
          ```

          > Redisson同时还为分布式可重入公平锁提供了异步执行的相关方法：

          ```java
          RLock fairLock = redisson.getFairLock("anyLock");
          fairLock.lockAsync();
          fairLock.lockAsync(10, TimeUnit.SECONDS);
          Future<Boolean> res = fairLock.tryLockAsync(100, 10, TimeUnit.SECONDS);
          ```

        - **读写锁（ReadWriteLock）**

          > 基于Redis的Redisson分布式可重入读写锁[`RReadWriteLock`](http://static.javadoc.io/org.redisson/redisson/3.4.3/org/redisson/api/RReadWriteLock.html) Java对象实现了`java.util.concurrent.locks.ReadWriteLock`接口。其中读锁和写锁都继承了[RLock](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#81-可重入锁reentrant-lock)接口。

          分布式可重入读写锁允许同时有多个读锁和一个写锁处于加锁状态。

          ```java
          RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
          // 最常见的使用方法
          rwlock.readLock().lock();
          // 或
          rwlock.writeLock().lock();
          ```

          **保证一定能读到最新数据，修改期间，写锁是一个排他锁（互斥、独享锁）。读锁是一个共享锁**
          **写锁没释放就必须等待**

          **读+读：相当于无锁，并发读，只会在redis中记录好，所有当前的读锁。他们都会同时加锁成功**

          **写+读：读必须等待写锁释放**
          **写+写：阻塞方式**
          **读+写：有读锁写也需要等待**
          **只要有写锁的存在，都必须等待**

          ```java
              @GetMapping("/write")
              @ResponseBody
              public String writeValue(){
                  String s = "";
                  RReadWriteLock readWriteLock = redisson.getReadWriteLock("rw-lock");
                  // 1.改数据加写锁
                  RLock writeLockLock = readWriteLock.writeLock();
                  try {
                      writeLockLock.lock();
                      System.out.println("写锁加锁成功..."+Thread.currentThread().getId());
                      s = UUID.randomUUID().toString();
                      Thread.sleep(30000);
                      redisTemplate.opsForValue().set("writeValue",s);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      writeLockLock.unlock();
                      System.out.println("写锁释放成功..."+Thread.currentThread().getId());
                  }
                  return s;
              }
          
              @GetMapping("/read")
              @ResponseBody
              public String readValue(){
                  String s = "";
                  RReadWriteLock readWriteLock = redisson.getReadWriteLock("rw-lock");
                  // 1.读数据加读锁
                  RLock readLock = readWriteLock.readLock();
                  readLock.lock();
                  System.out.println("读锁加锁成功..."+Thread.currentThread().getId());
                  try {
                      s = (String) redisTemplate.opsForValue().get("writeValue");
                      Thread.sleep(30000);
                  } catch (Exception e) {
                      e.printStackTrace();
                  } finally {
                      readLock.unlock();
                      System.out.println("读锁释放成功..."+Thread.currentThread().getId());
                  }
                  return s;
              }
          ```

        - **信号量（Semaphore）**

          > 基于Redis的Redisson的分布式信号量（[Semaphore](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphore.html)）Java对象`RSemaphore`采用了与`java.util.concurrent.Semaphore`相似的接口和用法。同时还提供了[异步（Async）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreAsync.html)、[反射式（Reactive）](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreReactive.html)和[RxJava2标准](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RSemaphoreRx.html)的接口。

          ```java
          RSemaphore semaphore = redisson.getSemaphore("semaphore");
          semaphore.acquire();
          //或
          semaphore.acquireAsync();
          semaphore.acquire(23);
          semaphore.tryAcquire();
          //或
          semaphore.tryAcquireAsync();
          semaphore.tryAcquire(23, TimeUnit.SECONDS);
          //或
          semaphore.tryAcquireAsync(23, TimeUnit.SECONDS);
          semaphore.release(10);
          semaphore.release();
          //或
          semaphore.releaseAsync();
          ```

          车库停车

          ```java
              /**
               * @Description: 车库停车
               * 3个车位
               * 信号量也可以用作分布式限流
               * @Param: []
               * @return: java.lang.String
               * @Author: Liuyunda
               * @Date: 2021/4/1
               */
              @GetMapping("/park")
              @ResponseBody
              public String park() throws InterruptedException {
                  RSemaphore park = redisson.getSemaphore("park");
                  // 获取一个信号，占一个车位
                  // 阻塞式，如果获取不到则一直获取
                  // park.acquire();
                  // 尝试获取一个信号
                  boolean tryAcquire = park.tryAcquire();
                  if (tryAcquire){
                      // 执行业务
                  }else {
                      return "信号量已满";
                  }
                  return "ok=>"+tryAcquire;
              }
              @GetMapping("/go")
              @ResponseBody
              public String go() throws InterruptedException {
                  RSemaphore park = redisson.getSemaphore("park");
                  // 释放一个信号,释放一个车位
                  park.release();
                  return "ok";
              }
          ```

        - **闭锁（CountDownLatch）**

          > 基于Redisson的Redisson分布式闭锁（[CountDownLatch](http://static.javadoc.io/org.redisson/redisson/3.10.0/org/redisson/api/RCountDownLatch.html)）Java对象`RCountDownLatch`采用了与`java.util.concurrent.CountDownLatch`相似的接口和用法。

          ```java
          RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
          latch.trySetCount(1);
          latch.await();
          
          // 在其他线程或其他JVM里
          RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
          latch.countDown();
          ```

          门卫锁门

          ```java
              /**
               * @Description: 放假，锁门
               * 所有班级走完才锁门
               * @Param: []
               * @return: java.lang.String
               * @Author: Liuyunda
               * @Date: 2021/4/1
               */
              @GetMapping("/lockDoor")
              @ResponseBody
              public String lockDoor() throws InterruptedException {
          
                  RCountDownLatch door = redisson.getCountDownLatch("door");
                  // 等待五个班
                  door.trySetCount(5);
                  // 等待闭锁都完成
                  door.await();
                  return "放假了！";
              }
          
              @GetMapping("/gogogo/{id}")
              @ResponseBody
              public String gogogo(@PathVariable("id")Long id){
                  RCountDownLatch door = redisson.getCountDownLatch("door");
                  // 计数减一
                  door.countDown();
                  return id+"班的人都走了....";
              }
          ```

   4. 缓存数据一致性

      > 锁的名字影响锁的粒度，越细越快
      >
      > 锁的粒度：具体缓存的是某个数据

      解决方案：

      1. 双写模式：同时修改缓存中的数据

         - 存在的问题：暂时性脏数据

           ![image-20210401232009836](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210401232009836.png)

         - 解决方案：1.对写数据库和写缓存整个操作加锁。2.给写缓存是设置过期时间

      2. 失效模式：删除缓存中的数据，等待下次主动查询进行更新

         - 存在的问题：脏数据

           ![image-20210401232911790](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210401232911790.png)

         - 解决方案：1.如果经常需要更改的数据，不建议放缓存，直接读数据库

      3. ![image-20210401233256991](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210401233256991.png) 

      4. ![image-20210401234013390](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210401234013390.png)

4. **Spring Cache**

   > https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache

   1. 简介

      - Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来同意不通的缓存技术，并支持使用JCache（JSR-107）注解简化我们开发。

      - Cache接口为缓存的组件规范定义，包含缓存的各种操作集合。Cache接口下Spring提供了各种xxxCache的实现。如RedisCache，EhCacheCache，ConcurrentMapCache等

      - 每次调用缓存需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

      - 使用Spring缓存抽象时我们需要关注一下两点：

        1. 确定方法需要被缓存以及他们的缓存策略
        2. 从缓存中读取之前缓存存储的数据

        ![springboot+redis缓存](https://gitee.com/SexJava/FigureBed/raw/master/static/springboot+redis缓存.png)

   2. 缓存注解

      - `@Cacheable`：触发将数据保存到缓存的操作。

        - 默认行为

          - 如果缓存中有，方法不调用
          - key默认自动生成；缓存的名字：：SimpleKey[]（自主生成的key值）
          - 缓存的value的值。默认使用jdk序列化机制。将序列化后的数据存到redis
          - 默认时间（TTL）-1

        - 自定义操作

          - 指定生成的缓存使用的key：key属性指定。接收一个SpEl表达式

            - | Name          | Location           | Description                                                  | Example                                                      |
              | :------------ | :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
              | `methodName`  | Root object        | 被调用方法的名称                                             | `#root.methodName`                                           |
              | `method`      | Root object        | 被调用的方法                                                 | `#root.method.name`                                          |
              | `target`      | Root object        | 被调用的目标对象                                             | `#root.target`                                               |
              | `targetClass` | Root object        | 被调用目标的类                                               | `#root.targetClass`                                          |
              | `args`        | Root object        | 用于调用目标的参数（作为数组）                               | `#root.args[0]`                                              |
              | `caches`      | Root object        | 运行当前方法的缓存的集合                                     | `#root.caches[0].name`                                       |
              | Argument name | Evaluation context | 任何方法参数的名称。 如果名称不可用（可能是由于没有调试信息），则参数名称在参数索引的where位置（从开头）下也可用。.`#a<#arg>``#arg``0` | `#iban` or (you can also use or notation as an alias).`#a0``#p0``#p<#arg>` |
              | `result`      | Evaluation context | 方法调用的结果（要缓存的值）。 仅在表达式，表达式（用于计算）或表达式（当是）中可用。 对于受支持的包装器（例如），是指实际对象，而不是包装器.`unless``cache put``key``cache evict``beforeInvocation``false``Optional``#result` | `#result`                                                    |

          - 指定缓存的数据的存活时间（过期时间）：配置文件中修改TTL，`spring.cache.redis.time-to-live=3600000`

          - 将数据保存为json格式

            - 默认配置

              ```java
              /**
               * Default {@link RedisCacheConfiguration} using the following:
               * <dl>
               * <dt>key expiration</dt>
               * <dd>eternal</dd>
               * <dt>cache null values</dt>
               * <dd>yes</dd>
               * <dt>prefix cache keys</dt>
               * <dd>yes</dd>
               * <dt>default prefix</dt>
               * <dd>[the actual cache name]</dd>
               * <dt>key serializer</dt>
               * <dd>{@link org.springframework.data.redis.serializer.StringRedisSerializer}</dd>
               * <dt>value serializer</dt>
               * <dd>{@link org.springframework.data.redis.serializer.JdkSerializationRedisSerializer}</dd>
               * <dt>conversion service</dt>
               * <dd>{@link DefaultFormattingConversionService} with {@link #registerDefaultConverters(ConverterRegistry) default}
               * cache key converters</dd>
               * </dl>
               *
               * @return new {@link RedisCacheConfiguration}.
               */
              public static RedisCacheConfiguration defaultCacheConfig() {
              	return defaultCacheConfig(null);
              }
              ```

            - 自定义配置类

              ```java
              /**
               * @Author Liuyunda
               * @Date 2021/4/6 23:04
               * @Email man021436@163.com
               * @Description: DOTO
               */
              @EnableConfigurationProperties(CacheProperties.class)
              @Configuration
              @EnableCaching
              public class MyCacheConfig {
              
                  // @Autowired
                  // CacheProperties cacheProperties;
                  @Bean
                  RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties){
                      RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
                      // config = config.entryTtl();
                      config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
                      config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
                      /**
                       * 原来和配置文件绑定的配置类是这样的
                       *   @ConfigurationProperties(prefix = "spring.cache")
                       *   public class CacheProperties
                       * 要让他生效
                       *   @EnableConfigurationProperties(CacheProperties.class)
                       */
                      CacheProperties.Redis redisProperties = cacheProperties.getRedis();
              
                      if (redisProperties.getTimeToLive() != null) {
                          config = config.entryTtl(redisProperties.getTimeToLive());
                      }
                      if (redisProperties.getKeyPrefix() != null) {
                          config = config.prefixKeysWith(redisProperties.getKeyPrefix());
                      }
                      if (!redisProperties.isCacheNullValues()) {
                          config = config.disableCachingNullValues();
                      }
                      if (!redisProperties.isUseKeyPrefix()) {
                          config = config.disableKeyPrefix();
                      }
                      return config;
                  }
              }
              ```

              

      - `@CacheEvict`：触发将数据从缓存删除的操作。失效模式

        - 进行多种操作：`@Caching`
     - 指定删除某个分区下的所有数据：`@CacheEvict(value = "category",allEntries = true)`
        - 存储同一类型的数据，都可以指定成同一个分区。分区名默认就是缓存的前缀

      - `@CachePut`：不影响方法执行的更新缓存。/双写模式

      - `@Caching`：组合多种缓存操作。

      - `@CacheConfig`：在类级别共享缓存的相同配置。

   3. 整合spring cache
   
      1. 导入依赖
   
         ```xml
         <dependency>
          <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-cache</artifactId>
      </dependency>
         ```

      2. 配置
   
      1. 自动配置：
   
         - `CacheAutoConfiguration`会导入`RedisCacheConfiguration`
            - `RedisCacheConfiguration`自动配置好缓存管理器`RedisCacheManager`
   
         2. 手动配置：
   
            ```properties
            spring.cache.type=redis
            spring.cache.redis.time-to-live=3600000
            #如果指定了前缀就用我们指定的前缀，如果没有就用缓存的名字作为前缀
            spring.cache.redis.key-prefix=CACHE_
         spring.cache.redis.use-key-prefix=true
            #是否缓存空值。防止缓存穿透
         spring.cache.redis.cache-null-values=true
            ```
   
   3. 测试缓存
   
      1. 开启缓存功能`@EnableCaching`
   
2. 只需要使用注解完成缓存操作
   
4. 原理
   
   `CacheAutoConfiguration`->导入`RedisCacheConfiguration`->自动配置了缓存管理器->`RedisCacheManager`->初始化所有的缓存->每个缓存决定使用什么配置->如果`redisCacheConfiguration`有就用已有的，没有就用默认配置。->想改缓存的配置只需要在容器中发一个`RedisCacheConfiguration即可`->就会应用到当前`RedisCacheManager`管理的所有缓存分区中。
   
   5. 不足
      
         1. 读模式
         - 缓存穿透：查询一个null数据。解决：缓存空数据`spring.cache.redis.cache-null-values=true`
            - 缓存击穿：大量并发进来同时查询一个正好过期的数据。解决：加锁
              - spring cache 默认没加锁
              - 加锁`@Cacheable(value = "category",key = "#root.method.name",sync = true)`
            - 缓存雪崩：大量的key同时过期。解决：加过期时间`spring.cache.redis.time-to-live=3600000`
         2. 写模式（缓存与数据库一致）
            1. 读写加锁（读多写少）
            2. 引入canal（感知到mysql的更新去更新数据库）
            3. 读多写多（直接去数据库查）
         3. 总结：
            1. 常规数据（读多写少，即时性，一致性要求不高的数据）完全可以使用spring-cache
            2. 特殊数据：特殊数据就要特殊设计
         
         
         
         
         
         
         
         

### 5.6.检索服务

1. 添加资源文件

2. 修改nginx配置

   ```bash
   server {
       listen       80;
       server_name mall.com *.mall.com;
   ```

3. 修改网关配置

   ```yml
           - id: mall_host_route
             uri: lb://mall-product
             predicates:
               - Host=mall.com
   
           - id: mall_search_route
             uri: lb://mall-search
             predicates:
               - Host=search.mall.com
   ```

4. 检索DSL语句

   模糊匹配，过滤（属性，分类，品牌，价格区间，库存），排序，分页，高亮，聚合分析

   ```json
   GET product/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "skuTitle": "华为"
             }
           }
         ],
         "filter": [
           {
             "term": {
               "catalogId": "225"
             }
           },
           {
             "terms": {
               "brandId": [
                 "4",
                 "1"
               ]
             }
           },
           {
             "nested": {
               "path": "attrs",
               "query": {
                 "bool": {
                   "must": [
                     {
                       "term": {
                         "attrs.attrId": {
                           "value": "11"
                         }
                       }
                     },
                     {
                       "terms": {
                         "attrs.attrValue": [
                           "abc",
                           "1"
                         ]
                       }
                     }
                   ]
                 }
               }
             }
           },
           {
             "term": {
               "hasStock": {
                 "value": "false"
               }
             }
           },
           {
             "range": {
               "skuPrice": {
                 "gte": 0,
                 "lte": 6000
               }
             }
           }
         ]
       }
     },
     "sort": [
       {
         "skuPrice": {
           "order": "desc"
         }
       }
     ],
     "from": 0,
     "size": 5,
     "highlight": {
       "fields": {"skuTitle": {}}, 
       "pre_tags": "<b style='color:red;'>",
       "post_tags": "</b>"
     }
   }
   ```

5. 修改映射（迁移）

   ```json
   PUT mall_product
   {
     "mappings": {
       "properties": {
         "attrs": {
           "type": "nested",
           "properties": {
             "attrId": {
               "type": "long"
             },
             "attrName": {
               "type": "keyword"
             },
             "attrValue": {
               "type": "keyword"
             }
           }
         },
         "brandId": {
           "type": "long"
         },
         "brandImg": {
           "type": "keyword"
         },
         "brandName": {
           "type": "keyword"
         },
         "catalogId": {
           "type": "long"
         },
         "catalogName": {
           "type": "keyword"
         },
         "hasStock": {
           "type": "boolean"
         },
         "hotScore": {
           "type": "long"
         },
         "saleCount": {
           "type": "long"
         },
         "skuId": {
           "type": "long"
         },
         "skuImg": {
           "type": "keyword"
         },
         "skuPrice": {
           "type": "keyword"
         },
         "skuTitle": {
           "type": "text",
           "analyzer": "ik_smart"
         },
         "spuId": {
           "type": "keyword"
         }
       }
     }
   }
   ```

   ```json
   POST _reindex
   {
     "source": {
       "index": "product"
     },
     "dest": {
       "index": "mall_product"
     }
   }
   ```

   修改索引常量

   ```java
   public class EsConstant {
       // sku在es中的索引
       public static final String PRODUCT_INDEX = "mall_product";
   }
   
   ```

   嵌入式属性，查询，聚合，分析都应该用嵌入式的

   ```json
   {
   "aggs": {
       "brand_agg": {
         "terms": {
           "field": "brandId",
           "size": 100
         },
         "aggs": {
           "brand_name_agg": {
             "terms": {
               "field": "brandName",
               "size": 1
             }
           },
           "brand_img_agg": {
             "terms": {
               "field": "brandImg",
               "size": 1
             }
           }
         }
       },
       "catalog_agg": {
         "terms": {
           "field": "catalogId",
           "size": 10
         },
         "aggs": {
           "catalog_name_agg": {
             "terms": {
               "field": "catalogName",
               "size": 10
             }
           }
         }
       },
       "attr_agg":{
         "nested": {
           "path": "attrs"
         },
         "aggs": {
           "attr_id_agg": {
             "terms": {
               "field": "attrs.attrId",
               "size": 10
             },
             "aggs": {
               "attr_name_agg": {
                 "terms": {
                   "field": "attrs.attrName",
                   "size": 10
                 }
               },
               "attr_value_agg":{
                 "terms": {
                   "field": "attrs.attrValue",
                   "size": 10
                 }
               }
             }
           }
         }
       }
     }
   }
   ```

   完整的DSL语句

   ```josn
   GET mall_product/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "skuTitle": "华为"
             }
           }
         ],
         "filter": [
           {
             "term": {
               "catalogId": "225"
             }
           },
           {
             "terms": {
               "brandId": [
                 "4",
                 "1"
               ]
             }
           },
           {
             "nested": {
               "path": "attrs",
               "query": {
                 "bool": {
                   "must": [
                     {
                       "term": {
                         "attrs.attrId": {
                           "value": "11"
                         }
                       }
                     },
                     {
                       "terms": {
                         "attrs.attrValue": [
                           "abc",
                           "1"
                         ]
                       }
                     }
                   ]
                 }
               }
             }
           },
           {
             "term": {
               "hasStock": {
                 "value": "false"
               }
             }
           },
           {
             "range": {
               "skuPrice": {
                 "gte": 0,
                 "lte": 6000
               }
             }
           }
         ]
       }
     },
     "sort": [
       {
         "skuPrice": {
           "order": "desc"
         }
       }
     ],
     "from": 0,
     "size": 5,
     "highlight": {
       "fields": {"skuTitle": {}}, 
       "pre_tags": "<b style='color:red;'>",
       "post_tags": "</b>"
     },
     "aggs": {
       "brand_agg": {
         "terms": {
           "field": "brandId",
           "size": 100
         },
         "aggs": {
           "brand_name_agg": {
             "terms": {
               "field": "brandName",
               "size": 1
             }
           },
           "brand_img_agg": {
             "terms": {
               "field": "brandImg",
               "size": 1
             }
           }
         }
       },
       "catalog_agg": {
         "terms": {
           "field": "catalogId",
           "size": 10
         },
         "aggs": {
           "catalog_name_agg": {
             "terms": {
               "field": "catalogName",
               "size": 10
             }
           }
         }
       },
       "attr_agg":{
         "nested": {
           "path": "attrs"
         },
         "aggs": {
           "attr_id_agg": {
             "terms": {
               "field": "attrs.attrId",
               "size": 10
             },
             "aggs": {
               "attr_name_agg": {
                 "terms": {
                   "field": "attrs.attrName",
                   "size": 10
                 }
               },
               "attr_value_agg":{
                 "terms": {
                   "field": "attrs.attrValue",
                   "size": 10
                 }
               }
             }
           }
         }
       }
     }
   }
   ```

   java 检索语句
   
   ```java
   		SearchSourceBuilder ssb = new SearchSourceBuilder();
           /**
            * 模糊匹配，过滤（属性，分类，品牌，价格区间，库存）
            */
           // bool query
           BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
           // must 模糊匹配
           if (StringUtils.isNotEmpty(searchParam.getKeyword())){
               boolQuery.must(QueryBuilders.matchQuery("skuTitle",searchParam.getKeyword()));
           }
           // filter 三级分类id
           if (searchParam.getCatalog3Id()!=null){
               boolQuery.filter(QueryBuilders.termQuery("catalogId",searchParam.getCatalog3Id()));
           }
           // filter 品牌id
           if (searchParam.getBrandId()!=null && searchParam.getBrandId().size()>0){
               boolQuery.filter(QueryBuilders.termsQuery("brandId",searchParam.getBrandId()));
           }
           // filter 属性
           if (searchParam.getAttrs()!=null && searchParam.getAttrs().size()>0){
               for (String attrString : searchParam.getAttrs()) {
                   BoolQueryBuilder nestedBoolQuery = QueryBuilders.boolQuery();
                   String[] s = attrString.split("_");
                   String attrId = s[0];
                   String[] attrValues = s[1].split(":");
                   nestedBoolQuery.must(QueryBuilders.termQuery("attrs.attrId",attrId));
                   nestedBoolQuery.must(QueryBuilders.termsQuery("attrs.attrValue",attrValues));
                   NestedQueryBuilder nestedQuery = QueryBuilders.nestedQuery("attrs",nestedBoolQuery, ScoreMode.None);
                   boolQuery.filter(nestedQuery);
               }
           }
   
   
           // filter 库存
           boolQuery.filter(QueryBuilders.termQuery("hasStock",searchParam.getHasStock()==1));
   
           // filter 价格区间
           if (StringUtils.isNotEmpty(searchParam.getSkuPrice())){
               RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("skuPrice");
               String[] s = searchParam.getSkuPrice().split("_");
               if (s.length==2){
                   rangeQuery.gte(s[0]).lte(s[1]);
               }else if (s.length==1){
                   if (searchParam.getSkuPrice().startsWith("_")){
                       rangeQuery.lte(s[0]);
                   }
                   if (searchParam.getSkuPrice().endsWith("_")){
                       rangeQuery.gte(s[0]);
                   }
               }
               boolQuery.filter(rangeQuery);
           }
           ssb.query(boolQuery);
           /**
            * 排序，分页，高亮
            */
           // 排序
           if (StringUtils.isNotEmpty(searchParam.getSort())){
               String sort = searchParam.getSort();
               String[] s = sort.split("_");
               SortOrder sortOrder = s[1].equalsIgnoreCase("asc")?SortOrder.ASC:SortOrder.DESC;
               ssb.sort(s[0], sortOrder);
           }
   
           // 分页
           ssb.from((searchParam.getPageNum()-1)*EsConstant.PRODUCT_PAGE_SIZE);
           ssb.size(EsConstant.PRODUCT_PAGE_SIZE);
   
           // 高亮
           if (StringUtils.isNotEmpty(searchParam.getKeyword())){
               HighlightBuilder highlightBuilder = new HighlightBuilder();
               highlightBuilder.field("skuTitle");
               highlightBuilder.preTags("<b style='color:red;'>");
               highlightBuilder.postTags("</b>");
               ssb.highlighter(highlightBuilder);
           }
           /**
            * 聚合分析
            */
           // 品牌聚合
           TermsAggregationBuilder brand_agg = AggregationBuilders.terms("brand_agg");
           brand_agg.field("brandId").size(50);
           // 品牌聚合子聚合
           brand_agg.subAggregation(AggregationBuilders.terms("brand_name_agg").field("brandName").size(1));
           brand_agg.subAggregation(AggregationBuilders.terms("brand_img_agg").field("brandImg").size(1));
           ssb.aggregation(brand_agg);
   
           // 分类聚合
           TermsAggregationBuilder catalog_agg = AggregationBuilders.terms("catalog_agg").field("catalogId").size(20);
           catalog_agg.subAggregation(AggregationBuilders.terms("catalog_name_agg").field("catalogName").size(1));
           ssb.aggregation(catalog_agg);
   
           // 属性聚合
           NestedAggregationBuilder attr_agg = AggregationBuilders.nested("attr_agg","attrs");
           TermsAggregationBuilder attr_id_agg = AggregationBuilders.terms("attr_id_agg").field("attrs.attrId").size(1);
           attr_id_agg.subAggregation(AggregationBuilders.terms("attr_name_agg").field("attrs.attrName").size(1));
           attr_id_agg.subAggregation(AggregationBuilders.terms("attr_value_agg").field("attrs.attrValue").size(50));
           attr_agg.subAggregation(attr_id_agg);
           ssb.aggregation(attr_agg);
   
           System.out.println(ssb.toString());
   ```
   
   postman 测试
   
   ![image-20210409211412943](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210409211412943.png)