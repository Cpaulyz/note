# SpringCloud(三)：Eureka服务注册与发现

[TOC]

## 1 Eureka简介

### 1.1 什么是Eureka

- Netflix在涉及Eureka时，遵循的就是API原则.
- Eureka是Netflix的有个子模块，也是核心模块之一。Eureka是基于REST的服务，用于定位服务，以实现云端中间件层服务发现和故障转移，服务注册与发现对于微服务来说是非常重要的，有了服务注册与发现，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了，功能类似于Dubbo的注册中心，比如Zookeeper.

### 1.2 基本原理

* SpringCloud 封装了Netflix公司开发的Eureka模块来实现服务注册与发现 (对比Zookeeper).

* Eureka采用了C-S的架构设计，EurekaServer作为服务注册功能的服务器，他是服务注册中心
* 而系统中的其他微服务，使用Eureka的客户端连接到EurekaServer并维持心跳连接。这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行，Springcloud 的一些其他模块 (比如Zuul) 就可以通过EurekaServer来发现系统中的其他微服务，并执行相关的逻辑

![image-20210203135514115](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203135514115.png)

### 1.3 组件

Eureka包含两个组件：Eureka Server和Eureka Clien**t**

* **Eureka Server**
	* Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样Eureka Server中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
	* Eureka Server本身也是一个服务，默认情况下会自动注册到Eureka注册中心。
* **Eureka Client**
	* Eureka Client是一个java客户端，用于简化与Eureka Server的交互，**客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后**，将会向Eureka Server发送心跳,**默认周期为30秒**，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个**服务节点移除(默认90秒)**。
	* Eureka Client分为两个角色，分别是：Application Service(Service Provider)和Application Client(Service Consumer)

### 1.4 三大角色

* Eureka Server：提供服务的注册与发现
* Service Provider：服务生产方，将自身服务注册到Eureka中，从而使服务消费方能狗找到
* Service Consumer：服务消费方，从Eureka中获取注册服务列表，从而找到消费服务

### 1.5 架构

![image-20210203135934387](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203135934387.png)

> Register(服务注册)：把自己的IP和端口注册给Eureka。
>
> Renew(服务续约)：发送心跳包，每30秒发送一次。告诉Eureka自己还活着。
>
> Cancel(服务下线)：当provider关闭时会向Eureka发送消息，把自己从服务列表中删除。防止consumer调用到不存在的服务。
>
> Get Registry(获取服务注册列表)：获取其他服务列表。
>
> Replicate(集群中数据同步)：eureka集群中的数据复制与同步。
>
> Make Remote Call(远程调用)：完成服务的远程调用。

## 2 构建单机Eureka Server

本文在[SpringCloud(二)：:Rest环境搭建](https://www.cnblogs.com/cpaulyz/p/14364353.html)的基础上进行构建

> 1. 导入依赖
> 2. 编写配置文件
> 3. 开启功能 enablexxxx
> 4. 配置类

### 2.1 eureka注册中心

创建新module

![image-20210203140103054](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203140103054.png)

修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-learning</artifactId>
        <groupId>com.cpaulyz</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-eureka-7001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.3.1.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

编写application.yml

```yml
eureka:
  instance:
    hostname: localhost # 服务端的实例名称
  client:
    register-with-eureka: false # 是否向eureka注册中心注册自己，因为本身是服务器，不需要注册
    fetch-registry: false # 如果为false，表示自己为注册中心；否则表示自己是服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
server:
  port: 7001
```

> 注意：修改defaultZone的依据在于service-url的源码
>
> ![image-20210203140715620](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203140715620.png)

书写启动类

```java
@SpringBootApplication
@EnableEurekaServer // 说明是服务端的启动类，可以接受其他服务注册
public class EurekaServer_7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_7001.class,args);
    }
}
```

启动即可

> 这里我报了个错误java.lang.TypeNotPresentException: Type javax.xml.bind.JAXBContext not present，解决办法是在module的maven中添加以下依赖
>
> ```xml
> <!--解决java.lang.TypeNotPresentException: Type javax.xml.bind.JAXBContext not present-->
> <dependency>
>     <groupId>javax.xml.bind</groupId>
>     <artifactId>jaxb-api</artifactId>
>     <version>2.3.0</version>
> </dependency>
> <dependency>
>     <groupId>com.sun.xml.bind</groupId>
>     <artifactId>jaxb-impl</artifactId>
>     <version>2.3.0</version>
> </dependency>
> <dependency>
>     <groupId>org.glassfish.jaxb</groupId>
>     <artifactId>jaxb-runtime</artifactId>
>     <version>2.3.0</version>
> </dependency>
> <dependency>
>     <groupId>javax.activation</groupId>
>     <artifactId>activation</artifactId>
>     <version>1.1.1</version>
> </dependency>
> ```

访问`localhost:7001`，可以看到以下页面

![image-20210203141645981](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203141645981.png)

## 3 服务注册

> 在springcloud-provider-dept-8001中操作

### 3.1 provider微服务注册

在pom.xml中新增

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.3.1.RELEASE</version>
        </dependency>
```

在application.yml中新增

```yml
# eureka配置，服务注册到哪里
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: spring-cloud-provider-dept8001 # 修改eureka上的描述信息
```

启动即可

重新访问`localhost:7001/`，可以看到以下页面

![image-20210203143936605](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203143936605.png)

### 3.2 eureka自我保护机制

一句话总结就是：**某时刻某一个微服务不可用，eureka不会立即清理，依旧会对该微服务的信息进行保存！**

- 默认情况下，当eureka server在一定时间内没有收到实例的心跳，便会把该实例从注册表中删除（**默认是90秒**），但是，如果短时间内丢失大量的实例心跳，便会触发eureka server的自我保护机制，比如在开发测试时，需要频繁地重启微服务实例，但是我们很少会把eureka server一起重启（因为在开发过程中不会修改eureka注册中心），**当一分钟内收到的心跳数大量减少时，会触发该保护机制**。可以在eureka管理界面看到Renews threshold和Renews(last min)，当后者（最后一分钟收到的心跳数）小于前者（心跳阈值）的时候，触发保护机制，会出现红色的警告：==EMERGENCY!EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT.RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEGING EXPIRED JUST TO BE SAFE==.从警告中可以看到，eureka认为虽然收不到实例的心跳，但它认为实例还是健康的，eureka会保护这些实例，不会把它们从注册表中删掉。
- 该保护机制的目的是避免网络连接故障，在发生网络故障时，微服务和注册中心之间无法正常通信，但服务本身是健康的，不应该注销该服务，如果eureka因网络故障而把微服务误删了，那即使网络恢复了，该微服务也不会重新注册到eureka server了，因为只有在微服务启动的时候才会发起注册请求，后面只会发送心跳和服务列表请求，这样的话，该实例虽然是运行着，但永远不会被其它服务所感知。所以，eureka server在短时间内丢失过多的客户端心跳时，会进入自我保护模式，该模式下，eureka会保护注册表中的信息，不在注销任何微服务，当网络故障恢复后，eureka会自动退出保护模式。自我保护模式可以让集群更加健壮。
- 但是我们在开发测试阶段，需要频繁地重启发布，如果触发了保护机制，则旧的服务实例没有被删除，这时请求有可能跑到旧的实例中，而该实例已经关闭了，这就导致请求错误，影响开发测试。所以，在开发测试阶段，我们可以把自我保护模式关闭，只需在eureka server配置文件中加上如下配置即可：`eureka.server.enable-self-preservation=false`【不推荐关闭自我保护机制】

> 更详细的介绍可以参考[Eureka自我保护机制](https://blog.csdn.net/wudiyong22/article/details/80827594)

### 3.3 拓展：开启微服务的actuator-info

> 一般在团队开发协作中使用

在pom.xml中添加

```xml
     	<!--actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

在application.yml中添加

```yml
# info配置
info:
  app.name: cpaulyz-springcloud
  company.name: cpaulyz
```

重新启动，访问`localhost:7001`，即eureka页面

![image-20210203144423613](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203144423613.png)



点击后，即可看到服务相关的info

![image-20210203150153819](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203150153819.png)

### 3.4 拓展：获取注册进来的微服务的信息

> 一般在团队开发协作中使用

在controller中添加以下，注意DiscoveryClient的包为`org.springframework.cloud.client.discovery.DiscoveryClient;`

```java
@RestController
@RequestMapping("/dept")
public class DeptController {
     ...
        
	 @Autowired
    DiscoveryClient client;

    @GetMapping("/discovery")
    public Object discovery(){
        // 获取微服务列表的清单
        List<String> services = client.getServices();
        System.out.println("discovery=>services:" + services);
        // 得到一个具体的微服务信息,通过具体的微服务id，applicaioinName；
        List<ServiceInstance> instances = client.getInstances("SPRINGCLOUD-PROVIDER-DEPT");
        for (ServiceInstance instance : instances) {
            System.out.println(
                    instance.getHost() + "\t" + // 主机名称
                            instance.getPort() + "\t" + // 端口号
                            instance.getUri() + "\t" + // uri
                            instance.getServiceId() // 服务id
            );
        }
        return this.client;
    }
}
```

在启动类上添加注解`@EnableDiscoveryClient`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient // 服务发现
public class DeptProvider_8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider_8001.class,args);
    }
}
```

请求url，即可获得服务信息

![image-20210203152519674](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203152519674.png)

控制台输出如下

![image-20210203152546259](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203152546259.png)

## 4 拓展：构建集群Eureka Server

> 在2中，我们构建的是单机的Eureka Server，为了做到高可用，实际开发中一般会用到集群，这里模拟集群Eureka Server的构建

### 4.1 构建注册中心集群

为了模拟在本地配置集群，需要修改一下hosts文件，新增以下条目

![image-20210203155029989](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203155029989.png)

按照之前构建单机Eureka Server的步骤，构建三个module，内容基本类似，这里不再赘述

![image-20210203155152200](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203155152200.png)

需要修改的地方有：三个module的application.yml

这里以7001为例

* 修改hostname
* 修改defaultZone，为另外两个eureka server

```yml
eureka:
  instance:
    hostname: eureka7001.com # 服务端的实例名称
  client:
    register-with-eureka: false # 是否向eureka注册中心注册自己，因为本身是服务器，不需要注册
    fetch-registry: false # 如果为false，表示自己为注册中心；否则表示自己是服务
    service-url:
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  server:
    enable-self-preservation: false
server:
  port: 7001
```

启动三个module即可

以7001为例，访问web页面可以看到另外两个集群

![image-20210203155446742](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203155446742.png)

### 4.2 向集群注册服务

修改springcloud-provider-dept-8001的application.yml，向集群注册

```yml
defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

然后访问注册中心，即可看到结果

![image-20210203155840602](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210203155840602.png)

## 5 CAP原则

CAP原则又称CAP定理，指的是在一个分布式系统中，Consistency（数据一致性）、 Availability（服务可用性）、Partition tolerance（分区容错性），三者不可兼得。CAP由Eric Brewer在2000年PODC会议上提出。该猜想在提出两年后被证明成立，成为我们熟知的CAP定理。

* **分布式系统CAP定理**

|                                  |                                                              |
| -------------------------------- | ------------------------------------------------------------ |
| 数据一致性(Consistency)          | 数据一致性(Consistency)也叫做数据原子性系统在执行某项操作后仍然处于一致的状态。在分布式系统中，更新操作执行成功后所有的用户都应该读到最新的值，这样的系统被认为是具有强一致性的。等同于所有节点访问同一份最新的数据副本。优点： 数据一致，没有数据错误可能。缺点： 相对效率降低。 |
| 服务可用性(Availablity)          | 每一个操作总是能够在一定的时间内返回结果，这里需要注意的是"一定时间内"和"返回结果"。一定时间内指的是，在可以容忍的范围内返回结果，结果可以是成功或者是失败。 |
| 分区容错性(Partition-torlerance) | 在网络分区的情况下，被分隔的节点仍能正常对外提供服务(分布式集群，数据被分布存储在不同的服务器上，无论什么情况，服务器都能正常被访问) |

* **定律：任何分布式系统只可同时满足二点，没法三者兼顾。**

|           |                                                              |
| --------- | ------------------------------------------------------------ |
| CA，放弃P | 如果想避免分区容错性问题的发生，一种做法是将所有的数据(与事务相关的)/服务都放在一台机器上。虽然无法100%保证系统不会出错，但不会碰到由分区带来的负面效果。当然这个选择会严重的影响系统的扩展性。 |
| CP，放弃A | 相对于放弃"分区容错性"来说，其反面就是放弃可用性。一旦遇到分区容错故障，那么受到影响的服务需要等待一定时间，因此在等待时间内系统无法对外提供服务。 |
| AP，放弃C | 这里所说的放弃一致性，并不是完全放弃数据一致性，而是放弃数据的强一致性，而保留数据的最终一致性。以网络购物为例，对只剩下一件库存的商品，如果同时接受了两个订单，那么较晚的订单将被告知商品告罄。 |

## 6 与zookeeper的区别

- Zookeeper 保证的是 CP —> 满足一致性，分区容错的系统，通常性能不是特别高
- Eureka 保证的是 AP —> 满足可用性，分区容错的系统，通常可能对一致性要求低一些

**Zookeeper保证的是CP**

 当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接收服务直接down掉不可用。也就是说，**服务注册功能对可用性的要求要高于一致性**。但zookeeper会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30-120s，且选举期间整个zookeeper集群是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因为网络问题使得zookeeper集群失去master节点是较大概率发生的事件，虽然服务最终能够恢复，但是，漫长的选举时间导致注册长期不可用，是不可容忍的。

**Eureka保证的是AP**

 Eureka看明白了这一点，因此在设计时就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时，如果发现连接失败，则会自动切换至其他节点，只要有一台Eureka还在，就能保住注册服务的可用性，只不过查到的信息可能不是最新的，除此之外，Eureka还有之中自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

- Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务
- Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上 (即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪

## 参考

[SpringCloud之Eureka注册中心原理及其搭建](https://www.cnblogs.com/jing99/p/11576133.html)

[CAP 定理的含义](http://www.ruanyifeng.com/blog/2018/07/cap.html)

[视频：狂神说Spring Cloud](https://www.bilibili.com/video/BV1jJ411S7xr)

[狂神说SpringCloud学习笔记](https://blog.csdn.net/weixin_43591980/article/details/106255122#t1)

