# SpringCloud笔记

## 1.父项目构建

### 1.1 dependencyManagement和dependencies区别

dependencyManagement能让所有在子项目中引用一个依赖而不用显式的列出版本号，maven会沿着父子层次向上走知道找到一个拥有dependencyManagement元素的项目，然后他就会使用这个dependencyManagement元素中的版本号。

dependencyManagement只是生命依赖，并不实现引入。

## 2.Eureka

### 2.1什么是服务治理

SpringCloud封装了NetFlix公司开发的Eureka模块来是脸服务治理

传统RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

### 2.2什么是服务注册与发现

Eureka采用了CS的设计架构，EurekaServer作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端链接到EurekaServer并维持心跳连接。这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息，比如服务地址通讯地址等以别名的方式注册到注册中心中，另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系（服务治理概念）。在任何RPC远程框架中，都会有一个注册中心（存放服务地址相关信息（接口地址））。

### 2.3Eureka的两个组件Eureka Server和EurekaClient

- EurekaServer提供服务注册服务

  各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

- EurekaClient通过注册中心进行访问

  是一个Java客户端，用与简化EurekaServer的交互，客户端同时也具备一个内置的、使用轮询（round-robin）负载算法的负载均衡器。在应用启动后，将会向EurekaServer发送心跳（默认周期为30 秒）。如果EurekaServer在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）。

### 2.4单机Eureka构建步骤

1. idea生成EurekaServer端服务注册中心（物业公司）
2. EurekaClient端cloud-provider-payment8001，将要注册进EurekaServer成为服务提供者provider（尚硅谷学校对外提供授课服务）
3. EurekaClient端cloud-consumer-order80，将要注册进EurekaServer成为服务消费者consumer（尚硅谷上课的同学）

### 2.5集群Eureka构建步骤

1. Eureka集群原理说明（互相注册，相互守望）

   1. 先启动Eureka注册中心

   2. 启动服务提供者payment支付服务

   3. 支付服务启动后会把自身信息（比如服务地址以别名方式注册进Eureka）

   4. 消费者order服务在需要调用接口时，使用服务别名去注册中心获取实际的Rpc远程调用地址

   5. 消费者获得地址后底层实际是利用HTTPClient技术实现远程调用

   6. 消费者获得服务地址后会缓存在本地jvm内存中，默认每间隔30秒更新一次服务调用地址

      **问题**：微服务RPC远程服务调用最核心的是什么

      **回答**：高可用，试想你的注册中心只有一个，他出故障了，就会导致整个微服务不可用。**所以采用**搭建Eureka注册中心集群，实现负载均衡+故障容错

2. EurekaServer集群环境构建步骤

   1. 参考7001
   2. 新建7002
   3. 改POM
   4. 修改映射配置
   5. 写YML
   6. 朱启东

3. 将支付服务8001微服务发布到上面2台Eureka集群配置中

4. 将订单服务80微服务发布到上面2台Eureka集群配置中

5. 测试01

   1. 先启动7001、7002
   2. 在启动8001、80

6. 支付服务提供者8001集群环境构建

   参考8001新建8002

7. 负载均衡

   - 使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
   - ![image-20200802215826490](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20200802215826490.png)

8. 测试02

   1. 先启动EurekaServer，7001、7002
   2. 在启动EurekaClient,8001/8002
   3. 在启动80
   4. localhost/consumer/payment/get/1
   5. 结果（轮询）
      - ![image-20200802220151548](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20200802220151548.png)
      - ![image-20200802220218018](https://gitee.com/SexJava/FigureBed/raw/master/static/image-20200802220218018.png)
   6. Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还具备负载均衡功能