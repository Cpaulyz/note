# SpringCloud(五)：Feign注解形式的服务调用

[TOC]



## 1 Feign简介

> Feign 本质上也是实现了 Ribbon，只不过后者是在调用方式上，为了满足一些开发者习惯的接口调用习惯
>
> 用于替换原本客户端下的各种RestTemplate操作

### 1.1 Feign是什么

Feign是声明式Web Service客户端，它让微服务之间的调用变得更简单，类似controller调用service。SpringCloud集成了Ribbon和Eureka，可以使用Feigin提供负载均衡的http客户端

**只需要创建一个接口，然后添加注解即可~**

Feign，主要是社区版，大家都习惯面向接口编程。这个是很多开发人员的规范。调用微服务访问两种方法

1. 微服务名字 【ribbon】
2. 接口和注解 【feign】

### 1.2 Feign能干什么

- Feign旨在使编写Java Http客户端变得更容易
- 前面在使用**Ribbon** + **RestTemplate**时，利用**RestTemplate**对Http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一个客户端类来包装这些依赖服务的调用。所以，**Feign**在此基础上做了进一步的封装，由他来帮助我们定义和实现依赖服务接口的定义，在Feign的实现下，我们只需要创建一个接口并使用注解的方式来配置它 (类似以前Dao接口上标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解)，即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon 时，自动封装服务调用客户端的开发量。

### 1.3 Feign与Ribbon

- 利用**Ribbon**维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡，而与**Ribbon**不同的是，通过**Feign**只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

- Feign 的一大特色就是声明式服务调用，对比于 Ribbon 的编程式实现，我们对两者进行比较

	|                              | Ribbon         | Feign                  |
	| ---------------------------- | -------------- | ---------------------- |
	| 使用对象                     | RestTemplate   | FeignService           |
	| 请求路由、参数、返回类型定义 | 编程实现       | 声明为接口             |
	| 调用特定服务                 | `xxxForObject` | 直接调用接口声明的方法 |



## 2 实战

> 在[SpringCloud(三)：Eureka与服务注册](https://www.cnblogs.com/cpaulyz/p/14372252.html#springcloud三：eureka与服务注册)的基础上，只需要修改客户端即可

### 2.1 修改springcloud-api

添加新依赖

```xml
        <!--Feign的依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
```

添加新的service包，并添加以下接口

```java
@Component
@FeignClient(value = "springcloud-provider-dept")
public interface DeptClientService {

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);

    @GetMapping("/dept/get/{id}")
    Dept get(@PathVariable("id") int id);

    @PostMapping("/dept/getAll")
    List<Dept> getAll();
}
```

> 注意：
>
> * `@FeignClient`注解中的value为服务名
> * 接口方法中的`@xxxMapping`应该与服务提供者的controller接口一致

其余无需修改，完成后的结构如下：

![image-20210208171412373](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208171412373.png)

### 2.2 创建springcloud-consumer-dept-feign-80

基本类似于springcloud-consumer-dept-80

![image-20210208171626990](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208171626990.png)

pom.xml依赖

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

    <artifactId>springcloud-consumer-dept-feign-80</artifactId>

    <dependencies>
        <!--Feign的依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <!--ribbon-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <!--eureka-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.3.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>com.cpaulyz</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

controller层

```java
@RestController
@RequestMapping("/consumer/dept")
public class DeptConsumerController {

    @Autowired
    DeptClientService deptClientService = null;

    @RequestMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        return deptClientService.get(id);
    }

    @RequestMapping("/add")
    public boolean add(Dept dept){
        return deptClientService.addDept(dept);
    }

    @RequestMapping("/getAll")
    public List<Dept> getAll(){
        return deptClientService.getAll();
    }

}
```

启动类

```java
@EnableEurekaClient
// feign客户端注解,并指定要扫描的包以及配置接口DeptClientService
@EnableFeignClients(basePackages = {"com.cpaulyz"})
@SpringBootApplication
public class FeignDeptConsumer_80 {
     public static void main(String[] args) {
        SpringApplication.run(FeignDeptConsumer_80.class,args);
    }
}
```

### 2.3 启动

启动服务提供者、eureka注册中心、feign消费者

![image-20210208171731831](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208171731831.png)

成功得到与Ribbon相似的效果

## 3 原理

总到来说，Feign的源码实现的过程如下：

- 首先通过@EnableFeignCleints注解开启FeignCleint
- 根据Feign的规则实现接口，并加@FeignCleint注解
- 程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。
- 当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate
- RequesTemplate在生成Request
- Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp
- 最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

## 参考

[视频：狂神说Spring Cloud](https://www.bilibili.com/video/BV1jJ411S7xr)

[狂神说SpringCloud学习笔记](https://blog.csdn.net/weixin_43591980/article/details/106255122#t1)

[Feign实现原理](https://blog.csdn.net/ciap37959/article/details/100619898)

[SpringCloud（三）之Feign实现负载均衡的使用](https://www.cnblogs.com/xiaowangbangzhu/p/10397037.html)