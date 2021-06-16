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
   
   返回结果封装
   
   ```java
   private SearchResult buildSearchResult(SearchResponse response, SearchParam searchParam) {
       SearchResult result = new SearchResult();
       // 1.返回的所有查询到的商品
       SearchHits hits = response.getHits();
       ArrayList<SkuEsModel> esModels = new ArrayList<>();
       if (hits.getHits() != null && hits.getHits().length > 0) {
           for (SearchHit hit : hits.getHits()) {
               String sourceAsString = hit.getSourceAsString();
               SkuEsModel esModel = JSON.parseObject(sourceAsString, SkuEsModel.class);
               if (StringUtils.isNotEmpty(searchParam.getKeyword())){
                   HighlightField skuTitle = hit.getHighlightFields().get("skuTitle");
                   String skuTitleHighlight = skuTitle.getFragments()[0].string();
                   esModel.setSkuTitle(skuTitleHighlight);
               }
               esModels.add(esModel);
           }
       }
       result.setProducts(esModels);
   
       // 2.当前商品涉及到到的所有属性信息
       ArrayList<SearchResult.AttrVo> attrVos = new ArrayList<>();
       ParsedNested attrAgg = response.getAggregations().get("attr_agg");
       ParsedLongTerms attrIdAgg = attrAgg.getAggregations().get("attr_id_agg");
       for (Terms.Bucket bucket : attrIdAgg.getBuckets()) {
           SearchResult.AttrVo attrVo = new SearchResult.AttrVo();
           // 属性id
           long attrId = bucket.getKeyAsNumber().longValue();
           attrVo.setAttrId(attrId);
           // 属性名
           String attrName = ((ParsedStringTerms) bucket.getAggregations().get("attr_name_agg")).getBuckets().get(0).getKeyAsString();
           attrVo.setAttrName(attrName);
           // 属性的所有值
           List<String> attrValues = ((ParsedStringTerms) bucket.getAggregations().get("attr_value_agg")).getBuckets().stream().map(item -> {
               String attrValue = item.getKeyAsString();
               return attrValue;
           }).collect(Collectors.toList());
           attrVo.setAttrValue(attrValues);
           attrVos.add(attrVo);
       }
       result.setAttrs(attrVos);
       // 3.当前商品涉及到的品牌信息
       ArrayList<SearchResult.BrandVo> brandVos = new ArrayList<>();
       ParsedLongTerms brandAgg = response.getAggregations().get("brand_agg");
       for (Terms.Bucket bucket : brandAgg.getBuckets()) {
           SearchResult.BrandVo brandVo = new SearchResult.BrandVo();
           // 品牌的id
           brandVo.setBrandId(bucket.getKeyAsNumber().longValue());
           // 品牌的名字
           brandVo.setBrandName(((ParsedStringTerms)bucket.getAggregations().get("brand_name_agg")).getBuckets().get(0).getKeyAsString());
           // 品牌的图片
           brandVo.setBrandImg(((ParsedStringTerms)bucket.getAggregations().get("brand_img_agg")).getBuckets().get(0).getKeyAsString());
           brandVos.add(brandVo);
       }
       result.setBrands(brandVos);
       // 4.分类信息
       ArrayList<SearchResult.CatalogVo> catalogVos = new ArrayList<>();
       ParsedLongTerms catalogAgg = response.getAggregations().get("catalog_agg");
       List<? extends Terms.Bucket> buckets = catalogAgg.getBuckets();
       for (Terms.Bucket bucket : buckets) {
           SearchResult.CatalogVo catalogVo = new SearchResult.CatalogVo();
           String catalogId = bucket.getKeyAsString();
           catalogVo.setCatalogId(Long.parseLong(catalogId));
   
           ParsedStringTerms catalogNameAgg = bucket.getAggregations().get("catalog_name_agg");
           String catalogName = catalogNameAgg.getBuckets().get(0).getKeyAsString();
           catalogVo.setCatalogName(catalogName);
           catalogVos.add(catalogVo);
       }
       result.setCatalogs(catalogVos);
   
       // 5.分页信息
       result.setPageNum(searchParam.getPageNum());
       long total = hits.getTotalHits().value;
       result.setTotal(total);
       int totalPages = (int) (total % EsConstant.PRODUCT_PAGE_SIZE == 0 ? total / EsConstant.PRODUCT_PAGE_SIZE : total / EsConstant.PRODUCT_PAGE_SIZE + 1);
       result.setTotalPages(totalPages);
       return result;
   }
   ```

### 5.7.异步线程池（略过）

### 5.8 认证服务(mall-auth-server)

1. 引入登录及注册页面（login.html 、reg.html）

2. 将静态资源上传者nginx服务器，mydata/nginx/html/static/login  及 mydata/nginx/html/static/reg

3. 修改springboot(2.2.6)版本与springcloud(Hoxton.SR9)版本与其他服务保持一致

4. 引入common依赖

   ```xml
   <dependency>
       <groupId>com.lyd.mall</groupId>
       <artifactId>mall-common</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <exclusions>
           <exclusion>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-boot-starter</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

5. 添加配置

   ```properties
   spring.application.name=mall-auth-server
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   server.port=20000
   ```

   

6. 主启动类添加注解启动服务注册与发现`@EnableDiscoveryClient`，远程调用服务`@EnableFeignClients`

7. `BCryptPasswordEncoder`密码加密

8. 社交登录（gitee为例）

   1. 登录授权后拿code

   2. 根据code拿access_token

   3. 根据access_token拿用户信息

      ![image-20210519222947177](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210519222947177.png)
   
   

### 5.9 Session共享问题

session原理

![image-20210520214627022](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210520214627022.png)

1. 不同服务，session不能共享问题
2. 同一服务，session不同步问题

session共享问题解决

1. session复制 

   1. 优点：web-server原生支持，只需要修改配置文件
   2. 缺点：session同步需要数据传输，占用大量网络带宽，降低了服务器群的业务才处理能力，任意一台web-server保存的数据都是所有web-server的session总和，收到内存限制无法水平扩展更多的web-server。大型分布式集群情况下，由于所有web-server都全量保存数据，所以此方案不可取。

2. 客户端存储

   1. 优点：服务器不需存储session，用户保存自己的session信息到cookie中，节省服务端资源
   2. 缺点：每次http请求，携带用户在cookie中的完整信息，浪费网络带宽。session的数据放在cookie中，cookie有长度限制4k，不能保存大量信息。session数据放在cookie中，存在泄漏，篡改，窃取等安全隐患

3. hash一致性

   1. 优点：
      1. 只需要改nginx配置，不需要修改应用代码
      2. 负载均衡，只要hash属性的值分布是均匀的，多台web-server的负载是均衡的
      3. 可以支持web-server水平扩展（session同步法是不行的，受内存限制）
   2. 缺点
      1. session还是存在web-server中的，所以web-server重启可能导致部分session丢失，影响业务，如部分用户需要重新登录
      2. 如果web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session
   3. ~但是以上缺点问题不是很大，因为session本身就是都有有效期的，所以这两种反向代理的方式可以使用

4. 统一存储

   1. 优点：
      1. 没有安全隐患
      2. 可以水平扩展，数据库/缓存水平切分即可
      3. web-server重启或扩容都不会有session丢失
   2. 缺点
      1. 增加了一次网络调用，并且需要修改应用代码；如将所有的getSession方法替换为从Redis查数据的方式。redis获取数据比内存慢很多
      2. 上面缺点可以用SpringSession完美解决

5. 不同服务，子域session共享

   ![image-20210520223700948](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210520223700948.png)

   1. 引入springSession依赖

      ```xml
      <!--整合springsession完成session共享-->
      <dependency>
          <groupId>org.springframework.session</groupId>
          <artifactId>spring-session-data-redis</artifactId>
      </dependency>
      ```

      

   2. 启动类开启`@EnableRedisHttpSession`注解

   3. 配置session数据存储类型

      ```properties
      # Session store type.
      spring.session.store-type=redis
      ```

      

   4. 修改redis序列化机制为json

      ```java
      @Bean
      public RedisSerializer<Object> springSessionDefaultRedisSerializer() {
          return new GenericJackson2JsonRedisSerializer();
      }
      ```

      

   5. 修改cookie的作用域为父作用域解决子域session不同享问题

      ```java
      @Bean
      public CookieSerializer cookieSerializer(){
          DefaultCookieSerializer defaultCookieSerializer = new DefaultCookieSerializer();
          defaultCookieSerializer.setDomainName("mall.com");
          defaultCookieSerializer.setCookieName("MALLSESSION");
          return defaultCookieSerializer;
      }
      ```


### 5.10. SpringSession核心原理

1. `@EnableRedisHttpSession`导入RedisHttpSessionConfiguration配置

   1. 给容器添加了一个组件

      1. `RedisIndexedSessionRepository`：redis操作session。session的增删改查封装类

   2. `springSessionRepositoryFilter`：session存储过滤器，每个请求过来都必须经过filter

   3. ```java
      	@Override
      	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
      			throws ServletException, IOException {
      		request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
      		// 包装原始请求对象
      		SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(request, response);
              // 包装原始响应对象
      		SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(wrappedRequest,
      				response);
      
      		try {
                  // 包装后的对象应用到了我们后面的整个执行链
      			filterChain.doFilter(wrappedRequest, wrappedResponse);
      		}
      		finally {
      			wrappedRequest.commitSession();
      		}
      	}
      ```

   4. 原始的request和response被包装成`SessionRepositoryRequestWrapper`和`SessionRepositoryResponseWrapper`

   5. 以后获取session。request.getSession();

   6. SessionRepositoryRequestWrapper.getSession()===>SessionRepository中获取到的

   7. 装饰者模式

### 5.11. 购物车

1. ​	RedirectAttributes
   1. addFlashAttribute()；将数据放在session中可以在页面取出，但是只能取一次
   2. addAttribute()；将数据放在请求参数中

### 5.12. 消息中间件-RabbitMQ

1. 应用场景
   1. 异步处理
   2. 应用解耦
   3. 流量控制
   
2. 概念

   1. 大多应用中，可通过消息服务中间件来提升系统异步通信，扩展解耦能力
   2. 消息服务中两个重要概念：消息代理（message broker）和目的地（destination），当消息发送者发送消息以后将由消息代理接管，消息代理保证消息传递到指定目的地
   3. 消息队列主要有两种形式的目的地
      1. 队列（queue）：点对点消息通信（point to point）
      2. 主题（topic）：发布（publish）/订阅（subscribe）消息通信
   4. 点对点式：
      - 消息发送者发送消息，消息代理将其放入一个队列中，信息接收者从队列中获取消息内容，消息读取后被移出队列
      - 消息只有唯一的发送者和接受者，但并不是说只能有一个接受者
   5. 发布订阅式：
      1. 发送者（发布者）发送消息到主题，多个接受者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时受到消息
   6. JMS（Java Message Service）Java消息服务
      1. 基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现
   7. AMQP（Advanced Message Queuing Protocol）
      1. 高级消息队列协议，也是一个消息代理的规范，兼容JMS
      2. RabbitMQ是AMQP的实现

3. JMS 与 AMQP对比

   |              | JMS（Java Message Service）                                  | AMQP（Advanced Message Queuing Protocol）                    |
   | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | 定义         | Java api                                                     | 网络线级协议                                                 |
   | 跨语言       | 否                                                           | 是                                                           |
   | 跨平台       | 否                                                           | 是                                                           |
   | Model        | 提供两种消息模型：<br />1 .Peer-2-Peer<br />2 .Pub/Sub       | 提供了五种消息模型：<br />1.direct exchange<br />2.fanout exchange<br />3.topic exchange<br />4.headers exchange<br />5.system exchange<br /><br />本质来讲，后四中和JMS的Pub/Sub模型没有太大差别，仅是在路由机制上做了更详细的划分 |
   | 支持消息类型 | 多种消息类型<br />TextMessage<br />MapMessage<br />BytesMessage<br />StreamMessage<br />ObjectMessage<br />Message(只有消息头和属性) | byte[],当实际应用时，有复杂的消息，可以将消息序列化后发送    |
   | 综合评价     | JMS定义了Java API层面的标准，在Java体系中，多个Client均可通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差 | AMQP定义了wire-level层的协议标准；天然具有跨平台，跨语言特性 |

4. **Spring支持**

   1. spring-jms提供了对jms的支持
   2. spring-rabbit提供了对AMQP的支持
   3. 需要ConnectionFactory的实现来链接消息代理
   4. 提供JmsTemplate、RabbitTemplate来发送消息
   5. @JmsListener（JMS）、@RabbitListener（AMQP）注解方法上监听消息代理发布的消息
   6. @EnableJms、@EnableRabbit开启支持

5. SpringBoot自动配置

   1. JmsAutoConfiguration
   2. RabbitAutoConfiguration

6. 市面上的MQ产品

   1. ActiveMQ、RabbitMQ、RocketMQ、Kafka

7. **RabbitMQ概念**

   1. RabbitMQ是一个有erlang开发的AMQP（Advanced Message Queuing Protocol）的开源实现

   2. 核心概念

      1. Message：消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则是由一系列的可选属性组成，这些属性包括    routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等
      2. Publisher：消息的生产者，也是一个向交换器发布消息的客户端应用程序
      3. Exchange：交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。Exchange有4种类型：direct（默认）、fanout、topic、headers，不同类型的Exchagne转发消息的策略有所区别
      4. Queue：消息队列，用来保存消息知道发送给消费者，它是消息的容器，也是消息的重点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
      5. Binding：绑定，用户消息队列和交换机之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。Exchange和Queue的绑定可以是多对多的关系
      6. Connection：网络连接，比如一个TCP连接
      7. Channel：信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真是的TCP连接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息，订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。
      8. Consumer：消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
      9. Virtual Host：虚拟主机，表示一批交换器，消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个virtual host本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。virtual host是AMQP概念的基础，必须在连接时指定，RabbitMQ默认的virtual host 是/
      10. Broker：表示消息队列服务器实体

      ![image-20210530155340089](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210530155340089.png)

8. Docker 安装RabbitMQ

   1. ```shell
      docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
      ```

      - 4369、25672（Erlang发现&集群端口）
      - 5671、5672（AMQP短裤）
      - 15672（Web管理后台端口）
      - 61613，61614（STOMP协议端口）
      - 1883，8883（MQTT协议端口）
   
9. RabbitMQ运行机制

   1. AMQP中的消息路由

      - AMQP中消息的路由过程和Java开发者熟悉的JMS存在一些差别，AMQP中增加了Exchange和Binding的角色。生产者吧消息发布到Exchange上，消息最终到达队列并被消费者接收，而Binding决定交换器的消息应该发送到那个队列。

   2. Exchange类型

      1. Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct（直接）、fanout（扇出）、topic（主题）、headers（头）。headers匹配AMQP消息的header而不是路由键，headers交换器和direct交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型。

         1. direct（直接）：消息中的路由键如果和binding key一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发routing key标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。他是完全匹配、单播的模式。

         2. fanout（扇出）：每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，知识简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，没太子网内的主机都获得了一份复制的消息。fanout类型转发消息是最快的。

         3. topic（主题）：topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。他将路由键和绑定键的字符串切分为单词，这些单词之间用点隔开。它同样也会识别两个通配符：#（匹配0个或多个单词）*（匹配一个单词）。

            ![image-20210531204826663](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210531204826663.png)

            ​	![image-20210531204850305](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210531204850305.png)

            ![image-20210531204933490](C:\Users\刘云达\AppData\Roaming\Typora\typora-user-images\image-20210531204933490.png)

            ​		![image-20210531205002547](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210531205002547.png)

10. RabbitMQ整合

    1. 引入spring-boot-starter-amqp

       1. ```xml
          <!--RabbitMQ-->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-amqp</artifactId>
          </dependency>
          ```

       2. 引入后，RabbitAutoConfiguration就会自动生效

       3. 给容器中自动配置了RabbitTemplate、AmqpAdmin、CachingConnectionFactory、RabbitMessagingTemplate

       4. @EnableRabbit

    2. application.yml

       1. ```properties
          spring.rabbitmq.host=192.168.56.10
          spring.rabbitmq.port=5672
          spring.rabbitmq.virtual-host=/
          ```

          

    3. 测试RabbitMQ

       1. AmqpAdmin：管理组件

          1. ```java
             // 使用Amqp进行创建exchange
             DirectExchange directExchange = new DirectExchange("hello-java-exchange",true,false);
             amqpAdmin.declareExchange(directExchange);
             log.info("exchange[{}]创建成功","hello-java-exchange");
             // 使用Amqp进行创建Queue
             Queue queue = new Queue("hello-java-queue",true,false,false);
             amqpAdmin.declareQueue(queue);
             log.info("queue[{}]创建成功","hello-java-queue");
             // 使用Amqp进行创建Binding
             Binding binding = new Binding("hello-java-queue", Binding.DestinationType.QUEUE,"hello-java-exchange","hello.java",null);
             amqpAdmin.declareBinding(binding);
             log.info("Binding[{}]创建成功","hello-java-Binding");
             ```

             

       2. RabbitTemplate：消息发送处理组件

          1. 发送消息

             ```java
                 @Autowired
                 RabbitTemplate rabbitTemplate;
             
                 @GetMapping("/sendMq")
                 public String sendMessage(@RequestParam(value = "num",defaultValue = "10")Integer num){
                     String message = "hello world";
                     // 如果发送的消息是个对象，会使用序列化机制，将对象写出去，对象必须实现Serializable
                     for (int i = 0; i < num; i++) {
                         if (i%2==0){
                             OrderReturnReasonEntity reasonEntity = new OrderReturnReasonEntity();
                             reasonEntity.setId(1L);
                             reasonEntity.setCreateTime(new Date());
                             reasonEntity.setName("qqqq-"+i);
                             reasonEntity.setStatus(1);
                             reasonEntity.setSort(1);
                             rabbitTemplate.convertAndSend("hello-java-exchange","hello.java", reasonEntity);
                         }else {
                             rabbitTemplate.convertAndSend("hello-java-exchange","hello.java", message);
                         }
             
                         // log.info("消息发送完成{}",reasonEntity.toString());
                     }
                     return "ok";
                 }
             ```

          2. 接收消息，监听消息使用@RabbitListener、@RabbitHandler

             ```java
             @RabbitListener(queues = {"hello-java-queue"})
             @Service("orderItemService")
             public class OrderItemServiceImpl extends ServiceImpl<OrderItemDao, OrderItemEntity> implements OrderItemService {
                 /**
                  * @Description: 监听队列
                  * 1.原生详细类型Message：消息头消息体
                  * 2.T 发送的消息的类型
                  * 3.Channel 当前传输数据的通道
                  *
                  * Queue:可以很多人都来监听。只要收到消息，队列删除消息。而且只能有一个收到此消息
                  *      场景
                  *          1.订单服务启动多个:同一个消息，只能有一个客户端收到
                  *          2.只有一个消息完全处理完，方法运行结束，就可以接收到下一个消息
                  *
                  * RabbitListener注解：类+方法上（监听哪些队列）
                  * RabbitHandler注解：方法上(重载区分不同的消息)
                  *
                  * @Param: []
                  * @return: void
                  * @Author: Liuyunda
                  * @Date: 2021/5/31
                  */
                 // @RabbitListener(queues = {"hello-java-queue"})
                 @RabbitHandler
                 public void receiveMessage(Message message, OrderReturnReasonEntity context, Channel channel){
                     // System.out.println("接收到消息，内容："+message+",类型："+message.getClass());
                     System.out.println("接收到消息，内容："+context);
                     // try { TimeUnit.SECONDS.sleep(3); }catch (InterruptedException e) { e.printStackTrace(); }
                     // System.out.println("消息处理完成");
                 }
             
                 @RabbitHandler
                 public void receiveMessage2(String context){
                     // System.out.println("接收到消息，内容："+message+",类型："+message.getClass());
                     System.out.println("接收到消息，内容："+context);
                     // try { TimeUnit.SECONDS.sleep(3); }catch (InterruptedException e) { e.printStackTrace(); }
                     // System.out.println("消息处理完成");
                 }
             }
             ```

       3. RabbitMQ消息确认机制-可靠抵达

          - 保证消息不丢失，可靠抵达，可以使用事务消息，性能下降250倍，为此引入确认机制

          - **publisher** confirmCallback 确认模式

          - **publisher** returnCallback 未投递到queue退回模式

          - **consumer** ack机制

            ![image-20210531222532873](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210531222532873.png)

          1. 可靠抵达-confirmCallback

             ```properties
             spring.rabbitmq.publisher-confirms=true
             #2.2.0之后correlated、simple、none
             spring.rabbitmq.publisher-confirm-type=correlated
             ```

             - 在创建connectionFactory的时候设置PublisherConfirms(true)选项，开启confirmCallback

             - CorrelationData：用来表示当前消息唯一性

             - 消息只要被broker接收到就会执行confirmCallback，如果是cluster（集群）模式，需要所有broker接收到才会调用confirmCallback

             - 被broker接收到只能表示message已经到达服务器，并不能保证能消息一定会被投递到目标queue里。所以需要用到接下来的returnCallback

               ```java
               @Configuration
               public class MyRabbitConfig {
               
                   @Autowired
                   RabbitTemplate rabbitTemplate;
               
               
                   @Bean
                   public MessageConverter messageConverter(){
                       return new Jackson2JsonMessageConverter();
                   }
               
               
                   /**
                    * @Description: 定制RabbitTemplate
                    * MyRabbitConfig对象创建完成以后，执行这个方法
                    * 1.服务器收到消息就回调
                    *      1.spring.rabbitmq.publisher-confirms=true
                    *      2.设置确认回调ConfirmCallback
                    * 2.消息正确抵达队列回调
                    * @Param: []
                    * @return: void
                    * @Author: Liuyunda
                    * @Date: 2021/5/31
                    */
                   @PostConstruct
                   public void initRabbitTemplate(){
                       // 设置确认回调
                       rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
                           /**
                            * @Description:只要消息抵达broker，ack就返回true
                            * @Param: [correlationData 当前消息的唯一关联数据（消息的唯一id）, ack 代表消息是否成功收到, cause 原因]
                            * @return: void
                            * @Author: Liuyunda
                            * @Date: 2021/5/31
                            */
                           @Override
                           public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                               System.out.println("correlationData:"+correlationData);
                               System.out.println("ack:"+ack);
                               System.out.println("cause:"+cause);
                           }
                       });
                   }
               
               }
               ```

          2. 可靠性抵达-ReturnCallback

             ```properties
             spring.rabbitmq.publisher-returns=true
             #只要抵达队列，以异步的方式优先回调我们这个returnConfirm
             spring.rabbitmq.template.mandatory=true
             ```

             - confirm 模式只能保证消息到达broker，不能保证消息准确投递到目标queue里。在有些业务场景下，我们需要保证消息一定要投递到目标queue里，此时就需要用到returnCallback退回模式

             - 这样如果未能投递到目标queue里将调用returnCallback，可以记录下详细到投递数据，定期的巡检或者自动纠错都需要这些数据

               ```java
               @Configuration
               public class MyRabbitConfig {
               
                   @Autowired
                   RabbitTemplate rabbitTemplate;
               
               
                   @Bean
                   public MessageConverter messageConverter(){
                       return new Jackson2JsonMessageConverter();
                   }
               
               
                   /**
                    * @Description: 定制RabbitTemplate
                    * MyRabbitConfig对象创建完成以后，执行这个方法
                    * 1.服务器收到消息就回调
                    *      1.spring.rabbitmq.publisher-confirms=true
                    *      2.设置确认回调ConfirmCallback
                    * 2.消息正确抵达队列回调
                    *      1.spring.rabbitmq.publisher-returns=true
                    *      2.spring.rabbitmq.template.mandatory=true(只要抵达队列，以异步的方式优先回调我们这个returnConfirm)
                    * @Param: []
                    * @return: void
                    * @Author: Liuyunda
                    * @Date: 2021/5/31
                    */
                   @PostConstruct
                   public void initRabbitTemplate(){
                       // 设置确认回调
                       rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
                           /**
                            * @Description:只要消息抵达broker，ack就返回true
                            * @Param: [correlationData 当前消息的唯一关联数据（消息的唯一id）, ack 代表消息是否成功收到, cause 原因]
                            * @return: void
                            * @Author: Liuyunda
                            * @Date: 2021/5/31
                            */
                           @Override
                           public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                               System.out.println("correlationData:"+correlationData);
                               System.out.println("ack:"+ack);
                               System.out.println("cause:"+cause);
                           }
                       });
                       // 设置消息抵达队列的确认回调
                       rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
                           /**
                            * @Description: 只要休息奥没有投递给指定的队列，就触发这个失败回调
                            * @Param: [message 投递失败的消息详细信息, replyCode 回复的状态码, replyText 回复的文本内容, exchange 当时这个消息发给哪个交换机, routingKey 当时这个消息用哪个路由键]
                            * @return: void
                            * @Author: Liuyunda
                            * @Date: 2021/5/31
                            */
                           @Override
                           public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                               System.out.println("message:"+message);
                               System.out.println("replyCode:"+replyCode);
                               System.out.println("replyText:"+replyText);
                               System.out.println("exchange:"+exchange);
                               System.out.println("routingKey:"+routingKey);
                           }
                       });
                   }
               
               }
               ```

          3. 可靠抵达-Ack消息确认机制
          
             1. 消费者获得到消息，成功处理，可以恢复Ack给Broker
          
                1. basic.ack 用于肯定确认；broker将移除此消息
                2. basic.nack 用于否定确认；可以指定broker是否丢弃此消息，可以批量
                3. basic.reject 用于否定确认；同上，但不能批量
          
             2. 默认，消息被消费者收到，就会从broker的queue中移除
          
             3. queue无消费者，消息依然会被存储。但是如果无法确定此消息是否被处理完成，或者成功处理。我们可以开启手动ack模式
          
                1. 消息处理成功，ack()，接收下一个消息，此消息broker就会移除
          
                2. 消息处理失败，nack()/reject()，重新发送给其他人进行处理，或者容错处理后ack
          
                3. 消息一直没有调用ack/nack方法，broker认为此消息正在被处理，不会投递给别人，此时客户端断开，消息不会被broker移除，会投递给别人
          
                   ```properties
                   # 手动ack消息
                   spring.rabbitmq.listener.simple.acknowledge-mode=manual
                   ```
          
                   ```java
                   @RabbitHandler
                   public void receiveMessage(Message message, OrderReturnReasonEntity context, Channel channel){
                       // System.out.println("接收到消息，内容："+message+",类型："+message.getClass());
                       System.out.println("接收到消息，内容："+context);
                       // try { TimeUnit.SECONDS.sleep(3); }catch (InterruptedException e) { e.printStackTrace(); }
                       // System.out.println("消息处理完成");
                       // channel内按顺序自增的
                       long deliveryTag = message.getMessageProperties().getDeliveryTag();
                       // 签收货物,非批量模式
                       try {
                           if (deliveryTag%2==0){
                               channel.basicAck(deliveryTag,false);
                               System.out.println("签收了");
                           }else {
                               // requeue = false 丢弃，=true发回服务器，服务器重新入队
                               channel.basicNack(deliveryTag,false,false);
                               // channel.basicReject();
                               System.out.println("拒签了");
                           }
                       } catch (IOException e) {
                           e.printStackTrace();
                       }
                   }
                   ```
          
                   **问题**：收到很多消息，自动回复给服务器ack，只有一个消息处理成功，系统宕机了。发生消息丢失
                   **解决**：消费者手动确认，只要没有明确告诉MQ，消息被接受，没有Ack。消息就一直是UnAcked状态，及时Consumer宕机，消息也不会丢失，会重新变为Ready，下一次有心的Consumer连接进来就发给他
                   **问题**：如何签收
                   **解决**：
          
                   ​	签收：channel.basicAck(deliveryTag,false);
                   ​    拒签：channel.basicNack(deliveryTag,false,false);

### 5.13 订单服务

- Feign远程调用丢失请求头问题  

  ![image-20210602222222879](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210602222222879.png)

  加上Feign远程调用的请求拦截器 

  ![image-20210602223822540](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210602223822540.png)

  ```java
  @Configuration
  public class MallFeignConfig {
      @Bean("requestInterceptor")
      public RequestInterceptor requestInterceptor(){
          return new RequestInterceptor() {
              @Override
              public void apply(RequestTemplate requestTemplate) {
                  // 1.RequestContextHolder拿到刚进来的这个请求的数据
                  ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                  // 老请求
                  HttpServletRequest request = requestAttributes.getRequest();
                  // 2.同步请求头数据，Cookie
                  String cookie = request.getHeader("Cookie");
                  // 给新请求同步了老请求的cookie
                  requestTemplate.header("Cookie",cookie);
              }
          };
      }
  }
  ```

- Feign异步情况丢失上下文问题

  异步编排用的线程不是同一个，不能获得ThreadLocal里面的信息，所以在异步编排的时候将老请求的内容同步过去

  ![image-20210603214842952](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210603214842952.png)

  ```java
  @Override
  public OrderConfirmVo confirmOrder() throws ExecutionException, InterruptedException {
      OrderConfirmVo confirmVo = new OrderConfirmVo();
      MemberResponseVo memberResponseVo = LoginUserInterceptor.loginUser.get();
  
      RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
      CompletableFuture<Void> getAddress = CompletableFuture.runAsync(() -> {
          //1.远程查询所有的收货地址列表
          RequestContextHolder.setRequestAttributes(requestAttributes);
          List<MemberAddressVo> receiveAddress = memberFeignService.getReceiveAddress(memberResponseVo.getId());
          confirmVo.setAddressVos(receiveAddress);
      }, executor);
  
      CompletableFuture<Void> getCart = CompletableFuture.runAsync(() -> {
          //2.远程查询购物车所有选中的购物项
          RequestContextHolder.setRequestAttributes(requestAttributes);
          List<OrderItemVo> cartItems = cartFeignService.getCurrentCartItems();
          confirmVo.setItems(cartItems);
      }, executor);
      //3.查询用户的积分
      Integer integration = memberResponseVo.getIntegration();
      confirmVo.setIntegration(integration);
  
      CompletableFuture.allOf(getAddress,getCart).get();
      // todo 防重令牌
      return confirmVo;
  }
  ```

- 接口幂等性问题

  - 什么是幂等性

    **接口幂等性就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的**，不会因为多次点击而产生了副作用，比如说支付场景，用户购买了商品支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条，这就没有保证接口的幂等性.

  - 哪些情况需要防止

    - 用户多次点击按钮
    - 用户页面回退再次提交
    - 微服务互相调用，由于网络问题，导致请求失败，Feign触发重试机制
    - 其他业务情况

  - 什么情况下需要幂等

    以SQL为例，有些操作时天然幂等的。

    select * from table where id = ? ,无论执行多少次都不会改变状态，是天然的幂等。

    update table set col1 =1 where col2 =2 ,无论执行成功多少次状态都是一致的，也是幂等操作。

    delete from user where userid = 1 ，多次操作，结果一样，具备幂等性

    insert into user(userid, name) values (1,'a')，如果userid为唯一主键，即重复操作上面的业务，只会插入一条用户数据，具备幂等性。

    ------

    update table set col1=col1+1 where col2 =2 ，每次执行的结果都会发生变化，不是幂等的

    insert into user(userid,name)values(1,'a')，如果userid不是主键，可以重复，那么上面业务多次操作，数据都会新增多条，不具备幂等性

  - 幂等解决方案

    1. token机制

       1. 服务端提供了发送token的接口，我们分析业务的时候，哪些业务是存在幂等问题的，就必须在执行业务前，先去获取token，服务器会把token保存到redis中。

       2. 然后调用业务接口请求时，把token携带过去，一般放在请求头头部。

       3. 服务器判断token是否存在redis中，表示第一次请求，然后删除token，继续执行业务。

       4. 如果判断token不在redis中，就表示是重复操作，直接返回重复标记给client，这样就保证了业务代码，不被重复执行。

          危险性：

          1. 先删除token还是后删除

             1. 先删除可能导致，业务确实没有执行，重试还带上之前token，由于防重设计导致，请求还是不能执行。
             2. 后删除可能导致没业务没处理成功，但是服务闪断，出现超时，没有删除token，别人继续重试，导致业务被执行两遍
             3. 我们最好设计为先删除token，如果业务调用失败，就重新获取token再次请求。

          2. Token 获取、比较和删除操作必须是原子性的

             1. redis.get(token),token.equals,redis.del(token)如果这两个操作不是原子，可能导致，高并发下，都get到同样的数据，判断都成功，继续业务并发执行

             2. 可以在redis使用lua脚本完成这个操作

                ```lua
                if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else retrun 0 end
                ```

    2. 各种锁机制

       1. 数据库悲观锁

          1. select * from xxx where id = 1 for update;

             悲观锁使用时，一般伴随事务一起使用，数据锁定时间可能会很长，需要根据实际情况选用。另外要注意的是，id字段一定是主键或者唯一索引，不然可能皂搓锁表的结果，处理起来会非常麻烦。

       2. 数据库乐观锁

          1. 这种方法适合在更新的场景中。

             update t_goods set count = count -1 ,version = version +1 where good_id = 2 and version = 1

             根据version版本，也就是在操作库存前，先获取当前商品的version版本号，然后操作的时候带上此版本号。我们梳理下，我们第一次操作库存时，得到version=1，调用库存服务version=2；但返回给订单服务出现了问题，订单服务又一次发起调用库存服务，当订单服务传回的version还是 1，再执行上面的sql语句时，就不会执行，因为version已经变为2了。where条件就不成立。这样就保证了不管调用几次，只会真正的处理一次。乐观锁主要使用于处理读多写少的问题

       3. 业务层分布式锁

          1. 如果多个机器可能在同一时间同时处理相同的数据，比如多台机器定时任务都拿到了相同数据处理，我们就可以加分布式锁，锁定此数据，处理完成后释放锁。获取到锁的必须先判断这个数据是否被处理过

    3. 各种唯一约束

       1. 数据库唯一约束
          1. 插入数据，应该按照唯一索引进行插入，比如订单号，相同的订单就不可能有两条记录插入。我们在数据库层面防止重复。这个机制是利用数据库主键唯一约束的特性，解决了在insert场景时幂等问题。但主键的要求不是自增的主键，这样就需要业务生成全局唯一的主键。如果是分库分表场景下，路由规则要保证相同请求下，落地在同一个数据库和同一表中，要不然数据库主键约束就不起效果了，因为是不同的数据库和表主键不相关。
       2. redis set 防重
          1. 很多数据需要处理，只能处理一次，比如我们可以计算数据的MD5将其放入redis的set，每次处理数据，先看这个MD5是否已经存在，存在就不处理

    4. 防重表

       1. 使用订单号orderNo作为去重表的唯一索引，把唯一索引插入去重表，再进行业务操作，且他们在同一个事务中。这个保证了重复请求时，因为去重表有唯一约束，导致请求失败，避免了幂等问题。这里要注意的是，去重表和业务表应该在同一库中，这样就保重了在同一个事务，及时业务操作失败了，也会把去重表的数据回滚，这个很好的保证了数据一致性。

    5. 全局请求唯一id

       1. 调用接口时，生成一个唯一id，redis将数据保存到集合中（去重），存在即处理过。可以使用nginx设置每一个请求的唯一id；

          ```
          proxy_set_header X-Request-Id $request_id;
          ```

- **分布式事务**

  ![image-20210609195244449](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210609195244449.png)

  - **远程服务假失败：远程服务其实成功了，但是由于网络问题没有返回或超时，导致本地事务回滚（订单回滚，库存却扣减了）**
  - **远程服务执行完成，下面的其他方法出现问题，导致已执行的远程请求，肯定不能回滚**

  

  

  

  

### 5.14.**本地事务**

1. **事务的基本性质**

   数据库事务的几个特性：原子性、一致性、隔离性或独立性和持久性，简称ACID

   - 原子性：一系列的操作整体不可拆分，要么同时成功，要么同时失败

   - 一致性：数据在事务的前后，业务整体一致。

     - 转账。A:1000,B:1000 A->B转200  A:800,B1200

   - 隔离性：事务之间相互隔离

   - 持久性：一旦事务成功，数据一定会存入数据库

     在以往的单体应用中，我们多个业务操作使用同一条连接操作不同的数据表，一旦有异常，我们可以很容易的整体回滚

2. **事务的隔离级别**

   1. read uncommitted（读未提交）

      该隔离级别的事务会读到其他未提交事务的数据，此现象也称为脏读

   2. read committed （读提交）

      一个事务可以读取另一个已提交的事务，多次读取会造成不一样的结果，此现象称为不可重复读，Oracle和Sql server 的默认隔离级别

   3. repeatable read （可重复读）

      该隔离级别是MySQL默认的隔离级别，在同一个事务里，select的结果是事务开始时时间点的状态，因此，同样的select操作读到的结果会是一致的，但是，会有幻读现象。MqSQL的innoDB引擎可以通过next-key locks机制来避免幻读

   4. serializable （序列化）

      在该隔离级别下事务都是串行顺序执行的，MySQL数据库的InnoDB引擎会给读操作隐式加一把读共享锁，从而避免脏读、不可重复度和幻读问题

3. **事务的传播行为**

   1. propagation_required：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置
   2. propagation_supports：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行
   3. propagation_mandatory：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常
   4. propagation_required_new：创建新事务，无论当前存不存在事务，都创建新事务
   5. propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
   6. propagation_never：以非事务方式执行，如果当前存在事务，则抛出异常
   7. propagation_nested：如果当前存在事务，则在嵌套事务内执行，如果当前没有事务，则执行与propagation_required类似的操作

4. SpringBoot事务关键点

   1. 事务的自动配置

      TransactionAutoConfiguration

   2. **事务的坑**

      1. 在同一个类里面，编写两个方法，内部调用的时候，会导致事务设置失效。原因是没有用到代理对象的缘故
      2. 解决：使用代理对象来调用事务方法
         1. 导入spring-boot-starter-aop->aspectJ
         2. @EnableTransactionManagement(proxyTargetClass = true)
         3. @EnableAspectJAutoProxy(exposeProxy = true) 开启aspectj动态代理功能。以后所有的动态代理都是aspectj创建的（即使没有接口也可以创建动态代理），对外暴露代理对象
         4. 本类互调用代理对象AopContext.currentProxy() 调用方法

1. **分布式事务**

   1. 为什么有分布式事务：分布式系统经常出现的异常：机器宕机、网络异常、信息丢失、信息乱序、数据错误、不可靠的TCP、存储数据丢失

   2. **CAP定理与BASE理论**

      1. CAP定理

         CAP原则又称CAP定理，指的是在一个分布式系统中

         - 一致性（Consistency）
           - 在分布式系统中的所有数据备份，在同一时刻是否是同样的值（等同于所有节点访问同一份最新的数据副本）
         - 可用性（Availability）
           - 在集群中 一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
         - 分区容错性（Partition tolerance）
           - 大多数分布式系统都分布在多个子网络。每个子 网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。比如，一台服务器放在中国，另一台服务器放在美国，这就是两个区，他们之前可能无法通信。

         CAP原则指的是，这三个要素最多只能同时实现两点，**不能三者兼顾**

         一般来说，分区容错无法避免，因此可以认为CAP的P总是成立的。CAP定理告诉我们，剩下的C和A无法同时做到

         分布式系统中实现一致性的Raft算法

         [Raft (thesecretlivesofdata.com)](http://thesecretlivesofdata.com/raft/)

      2. 面临的问题

         对于多数大型互联网应用的场景，主机众多，部署分散，而且现在集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到99.9999%（N个9），即保证P和A，舍弃C

      3. Base定理

         是对CAP理论的延伸，思想史即使无法做到强一致性（CAP的一致性就是强一致性），但可以采用适当的若已执行，即最终一致性

         Base是指

         - 基本可用
           - 基本可用是指分布式系统在存现故障的时候，允许损失部分可用性（例如响应时间、功能上的可用性）。需要注意的是，基本可用绝不等价于系统不可用
             - 响应时间上的损失：正常情况下搜索引擎需要在0.5秒之内返回给用户响应的查询结果，但是由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到1-2秒
             - 功能上的损失：购物网站在购物高峰时，为了保护系统的稳定性，部分消费者可能会被引导到一个降级页面。
         - 软状态
           - 软状态是指允许系统存在中间状态，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据会有多个副本，允许不同副本同步的延时就是软状态的体现。mysql replication的异步复制也是一种体现。
         - 最终一致性
           - 最终一致性是指系统中的所有数据副本经过一定时间后，最终能达到一致的状态。弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况

      4. 强一致性、弱一致性、最终一致性

         从客户端角度，多进程并发访问时，更新过的数据在不同进程如何获取的不同策略，决定了不同的一致性。对于关系型数据库，要求更新过的数据能被后续的访问都看到，这是强一致性。如果能容忍后续的部分或者全部访问不到，则是弱一致性。如果经过一段时间后要求能访问到更新后的数据，则是最终一致性
      
   3. 分布式事务几种方案

      1. 2PC模式

         数据库支持的2PC【2 phase commit 二阶段提交】，又叫做XA Transactions

         MySQL 从5.5版本开始支持，SQL Server 2005开始支持，Oracle 7开始支持

         其中，XA是一个两阶段提交协议，该协议分为以下两个阶段：

         1. 第一阶段：事务协调器要求每个涉及到事务的数据库预提交（precommit）此操作，并反映是否可以提交，
         2. 第二阶段：事务协调器要求每个数据库提交数据。其中如果有任何一个数据库否决此次提交，那么所有数据库都会被要求回滚他们在此事务中的那部分信息

         - XA协议比较简单，而且一旦商业数据库实现了XA协议，使用分布式事务的成本也比较低
         - **XA性能不理想**，特别是在交易下单链路，往往并发量很高，XA无法满足高并发场景
         - XA目前在商业数据库支持的比较理想，在mysql数据库中支持的不太理想，mysql的XA实现，没有记录prepare阶段日志，准备切换回导致主库与备库数据库数据不一致。
         - 许多nosql也没有支持XA，这让XA的应用场景变得非常狭隘。
         - 也有3PC，引入了超时机制（无论协调者还是参与者，在向对方发送请求后，若长时间未收到回应则做出相应处理）

      2. 柔性事务-TCC事务补偿性方案

         - 刚性事务：遵循ACID原则，强一致性。

         - 柔性事务：遵循BASE理论，最终一致性，柔性事务与刚性事务不同，柔性事务允许一定时间内，不同节点的数据不一致，但要求最终一致

           ![image-20210610194817593](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210610194817593.png)

           一阶段prepare行为，调用自定义的prepare逻辑

           二阶段commit行为，调用自定义的commit逻辑

           二阶段rollback行为。调用自定义的rollback逻辑

           所谓TCC模式，是指支持把自定义的分布事务纳入到全局事务的管理中

      3. 柔性事务-最大努力通知型方案

         按规律进行通知，不保证数据一定能通知成功，但会提供可查询操作接口进行核对。这种方案主要用在与第三方系统通讯时，比如：调用微信或支付宝支付后的支付结果通知。这种方案也是结合MQ进行实现，例如：通过MQ发送http请求，设置最大通知次数。达到通知次数后即不再通知。

         案例：银行通知、商户通知等（各大交易业务平台间的商户通知：多次通知、查询校对、对账文件），支付宝的支付成功异步回调

      4. 柔性事务-可靠消息+最终一致性方案（异步确保型）

         实现：业务处理服务在业务事务提交之前，向实时消息服务发送消息，实时消息服务只记录消息数据，而不是真正的发送。业务处理服务在业务事务提交之后，向实时消息服务确认发送，只有在得到确认发送指令后，实时消息服务才会真正发送

### 5.15. SpringCloud Alibaba Seata（高并发场景下不考虑）

seata控制分布式事务

1. 每一个微服务先必须创建`UNDO_LOG`表

2. 安装事务协调器（TC）

   [seata-server]: https://github.com/seata/seata/releases

3. 整合

   1. 导入依赖

      ```xml
      <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-alibaba-seata</artifactId>
      </dependency>
      ```

   2. 启动seata服务器

      1. registry.conf ：注册中心配置 registry.type 指定为nacos

   3. 所有想要用到分布式事务的微服务，使用seata DataSourceProxy 代理数据源

      ```java
      @Configuration
      public class MySeataConfig {
      
          @Autowired
          DataSourceProperties dataSourceProperties;
          @Bean
          public DataSource dataSource(){
              HikariDataSource build = dataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
              return new DataSourceProxy(build);
          }
      
      }
      ```

   4. 每个微服务，都必须导入file.conf和regis.conf

      1. file.conf 修改vgroup_mapping.{application.name}-fescar-service-group="default"

   5. 给分布式大事务的入口标注@GlobalTransactional，每一个小事务标注@Transactional

### 5.16. RabbitMQ延时队列（实现定时任务）

场景：比如未付款的 订单，超过一定时间后，系统自动取消订单并释放占有物品

常用的解决方案：Spring的schedule定时任务轮询数据库

- 缺点：小猴系统内存，增加数据库的压力、存在较大的时间误差

解决：RabbitMQ的消息TTL和死信Exchange结合

1. 消息的TTL（Time To Live）
   - 消息的TTL就是消息的存活时间
   - RabbitMQ可以对队列和消息分别设置TTL
     - 对队列设置就是队列没有消费者连着的保留时间，也可以对每一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就死了，称之为死信
     - 如果队列设置了，消息也设置了，那么会取小的。所以一个消息如果被路由到不通的队列中，这个消息死亡的时间有可能不一样（不同的队列设置）。这里单讲单个的TTL，因为它才是实现延迟任务的关键。可以通过设置消息的expiration字段或者x-message-ttl属性来设置时间，两者的效果是一样的。
2. Dead Letter Exchange （DLX）
   1. 一个消息在满足如下条件，会进入死信路由，记住这里是个路由而不是队列，一个路由可以对应很多队列（什么是死信）
      1. 一个消息被Consumer拒收了，并且reject方法的参数里requeue是false。也就是说不会被再次放在队列里被其他消费者使用。（basic.reject/basic.nack) requeue = false
      2. 上面的消息的TTL到了，消息过期了
      3. 队列的长度限制满了。排在前面的消息会被丢弃或扔到死信路由上
   2. Dead Letter Exchange 其实就是一种普通的exchange，和创建其他的exchange没有两样。只是在某一个设置Dead Letter Exchange 的队列中有消息过去了，会自动触发消息的转发，发送到Dead Letter Exchange中去。
   3. 我们既可以控制消息在一段时间后变成死信，有可以控制变成死信的消息被路由到某一个指定的交换机，结合二者，其实就可以实现一个延时队列

![image-20210613221654243](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210613221654243.png)

![image-20210613221939477](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20210613221939477.png)

### 5.17. RabbitMQ如何保证消息可靠性

1. 消息丢失

   - 消息发送出去，由于网络问题没有抵达服务器
     - 做好容错方法，发送消息可能会网络失败，失败后要有重试机制，可以记录导数据库，采用定期扫描重发的方式
     - 做好日志记录，每个消息状态是否被服务器收到都应该记录
     - 做好定期重发，如果消息没有发送成功，定期去数据库扫描未成功的消息进行重发
   - 消息抵达Broker，Broker要将消息写入磁盘（持久化）才算成功。此时Broker尚未持久化完成，宕机。
     - publisher也必须加入确认回调机制，确认成功的消息，修改数据库消息状态
   - 自动ACK的状态下。消费者收到消息，但没来的及消费然后宕机
     - 一定开启手动ACK，消费成功才移除，失败或者没来得及处理就noAck并重新入队

   做好消费确认机制（publisher，consumer手动确认），每一个发送的消息都在数据库做好记录，定期将失败的消息再次发送

2. 消息重复

   - 消息消费成功，事务已经提交，ack时，机器宕机。导致没有ack成功，Broker的消息重新有unack变为了ready，并发送给其他消费者
   - 消息消费失败，由于重试机制，自动又将消息发送出去
   - 成功消费，ack宕机，消息由unack变为ready，Broker又重新发送
     - 消费者的业务消费接口应该设计为幂等性的。比如扣库存有工作单的状态标志
     - 使用防重表，发送消息每一个都有业务的唯一标识，处理过就不用处理
     - RabbitMQ的每一个消息都有redelivered字段，可以获取是否是被重新投递过来的，而不是第一次投递过来的

3. 消息积压

   1. 消费者宕机积压
   2. 消费者消费能力不足积压
   3. 发送者发送流量太大
      1. 上线更多的消费者，进行正常消费
      2. 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理
