# SpringCloud(六)：Hystrix之服务熔断、服务降级

[TOC]



## 1 Hystrix介绍

> * Hystrix是一个应用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等
>
> * Hystrix 能够保证在一个依赖出问题的情况下，不会导致整个体系服务失败，避免级联故障，以提高分布式系统的弹性

### 1.1 解决问题：服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”，如果扇出的链路上**某个微服务的调用响应时间过长，或者不可用**，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”。

例如，正常情况下的请求是这样的

![image-20210208173752892](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208173752892.png)

当其中有一个系统有延迟时，它可能阻塞整个用户请求：

![image-20210208173806911](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208173806911.png)

随着大容量通信量的增加，单个后端依赖项的潜在性会导致所有服务器上的所有资源在几秒钟内饱和。

应用程序中通过网络或客户端库可能导致网络请求的每个点都是潜在故障的来源。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，从而备份队列、线程和其他系统资源，从而导致更多跨系统的级联故障。

![image-20210208173821049](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208173821049.png)

因此，对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几十秒内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障，**这些都表示需要对故障和延迟进行隔离和管理，以达到单个依赖关系的失败而不影响整个应用程序或系统运行**。

我们需要，**弃车保帅**，例如使用断路器：

> “断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控 (类似熔断保险丝) ，向调用方返回一个服务预期的，可处理的备选响应 (FallBack) ，而不是长时间的等待或者抛出调用方法无法处理的异常，这样就可以保证了服务调用方的线程不会被长时间，不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

### 1.2 设计原则

- 防止任何单个依赖项耗尽所有容器（如Tomcat）用户线程。
- 甩掉包袱，快速失败而不是排队。
- 在任何可行的地方提供回退，以保护用户不受失败的影响。
- 使用隔离技术（如隔离板、泳道和断路器模式）来限制任何一个依赖项的影响。
- 通过近实时的度量、监视和警报来优化发现时间。
- 通过配置的低延迟传播来优化恢复时间。
- 支持对Hystrix的大多数方面的动态属性更改，允许使用低延迟反馈循环进行实时操作修改。
- 避免在整个依赖客户端执行中出现故障，而不仅仅是在网络流量中。

### 1.3 Hystrix能做什么

- 服务降级
- 服务熔断
- 服务限流
- 接近实时的监控
- …

## 2 服务熔断

### 2.1 什么是服务熔断

> 熔断机制是赌赢雪崩效应的一种微服务链路保护机制。

当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，**进而熔断该节点微服务的调用，快速返回错误的响应信息**。

检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阀值缺省是**5秒内20次调用失败，就会启动熔断机制**。

熔断机制的注解是：`@HystrixCommand`

启动服务熔断的注解是：`@EnableCircuitBreaker`

服务熔断解决如下问题：

- 当所依赖的对象不稳定时，能够起到快速失败的目的；
- 快速失败后，能够根据一定的算法动态试探所依赖对象是否恢复。

### 2.2 demo

> 构建一个新的module，名为springcloud-provider-dept-hystrix-8001

项目结构：

![image-20210209155823403](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209155823403.png)

直接把springcloud-provider-dept-8001的src复制过去做修改

pom.xml

```xml
        <!--导入Hystrix依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
```

application.yml

```yml
server:
  port: 8001

# mybatis配置
mybatis:
  type-aliases-package: com.cpaulyz.PO
  mapper-locations: classpath:mybatis/mapper/*.xml

#  spring配置
spring:
  application:
    name: springcloud-provider-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springcloud?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
    username: root
    password: 123456


# eureka配置，服务注册到哪里
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
#      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: spring-cloud-provider-dept-hystrix-8001 # 修改eureka上的描述信息

# info配置
info:
  app.name: cpaulyz-springcloud
  company.name: cpaulyz
```

修改DeptController，关键注解：`@HystrixCommand`

```java
@RestController
@RequestMapping("/dept")
public class DeptController {

    @Autowired
    DeptService deptService;

    @GetMapping("/get/{id}")
    @HystrixCommand(fallbackMethod = "hystrixGet")
    public Dept get(@PathVariable("id") int id) {
        Dept dept =  deptService.queryById(id);
        if(dept==null){
            throw new RuntimeException("id=>"+id+"不存在");
        }
        return dept;
    }

    public Dept hystrixGet(@PathVariable("id") int id) {
        Dept dept = new Dept()
                .setDNo(id)
                .setDName("id=>"+id+"不存在")
                .setDName("no database");
        return dept;
    }
}
```

修改启动类，关键注解：`@EnableCircuitBreaker`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient // 服务发现
@EnableCircuitBreaker // 添加对熔断的支持
public class DeptProviderHystrix_8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8001.class,args);
    }
}
```

### 2.3 体验

启动7001Eureka中心、8001服务提供者、80客户端

对比一下添加服务熔断前后的代码与运行效果

#### 前（无熔断机制）：

```java
@RestController
@RequestMapping("/dept")
public class DeptController {

    @Autowired
    DeptService deptService;
    
    @GetMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        Dept dept =  deptService.queryById(id);
        if(dept==null){
            throw new RuntimeException("id=>"+id+"不存在");
        }
        return dept;
    }
    
    ...
}

```

请求一个不存在的id

![image-20210209161619526](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209161619526.png)

#### 后（有熔断机制）：

```java
@RestController
@RequestMapping("/dept")
public class DeptController {

    @Autowired
    DeptService deptService;

    @GetMapping("/get/{id}")
    @HystrixCommand(fallbackMethod = "hystrixGet")
    public Dept get(@PathVariable("id") int id) {
        Dept dept =  deptService.queryById(id);
        if(dept==null){
            throw new RuntimeException("id=>"+id+"不存在");
        }
        return dept;
    }

    public Dept hystrixGet(@PathVariable("id") int id) {
        Dept dept = new Dept()
                .setDNo(id)
                .setDName("id=>"+id+"不存在")
                .setDName("no database");
        return dept;
    }
}
```

请求一个不存在的id

![image-20210209161443305](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209161443305.png)

## 3 服务降级

### 3.1 什么是服务降级

服务降级是指 当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理，或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作。说白了，**就是尽可能的把系统资源让给优先级高的服务**。

资源有限，而请求是无限的。如果在并发高峰期，不做服务降级处理，一方面肯定会影响整体服务的性能，严重的话可能会导致宕机某些重要的服务不可用。所以，一般在高峰期，为了保证核心功能服务的可用性，都要对某些服务降级处理。比如当双11活动时，把交易无关的服务统统降级，如查看蚂蚁深林，查看历史订单等等。

服务降级主要用于什么场景呢？当整个微服务架构整体的负载超出了预设的上限阈值或即将到来的流量预计将会超过预设的阈值时，为了保证重要或基本的服务能正常运行，可以将一些 不重要 或 不紧急 的服务或任务进行服务的 延迟使用 或 暂停使用。

降级的方式可以根据业务来，可以延迟服务，比如延迟给用户增加积分，只是放到一个缓存中，等服务平稳之后再执行 ；或者在粒度范围内关闭服务，比如关闭相关文章的推荐。

> ![image-20210209161921557](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209161921557.png)
>
> 例如，**当某一时间内服务A的访问量暴增，而B和C的访问量较少，为了缓解A服务的压力，这时候需要B和C暂时关闭一些服务功能，去承担A的部分服务，从而为A分担压力，叫做服务降级**。
>
> 如果客户访问C服务，直接告诉他不可以访问！
>
> 比如双十一的时候，淘宝关闭退款服务

### 3.2 demo

> 直接在springcloud-api和springcloud-consumer-dept-feign-80中进行修改即可

springcloud-api中添加类并实现FallbackFactory

```java
/**
 * @Description: Hystrix服务降级 ~
 */
@Component
public class DeptClientServiceFallBackFactory implements FallbackFactory {
    @Override
    public DeptClientService create(Throwable cause) {
        return new DeptClientService() {
            @Override
            public boolean addDept(Dept dept) {
                return false;
            }
            @Override
            public Dept get(int id) {
                return new Dept()
                        .setDNo(id)
                        .setDName("id=>" + id + "没有对应的信息，客户端提供了降级的信息，这个服务现在已经被关闭")
                        .setDbSource("没有数据~");
            }
            @Override
            public List<Dept> getAll() {
                return null;
            }
        };
    }
}
```

springcloud-api中修改Service接口，在Feign注解中添加,fallbackFactory = DeptClientServiceFallBackFactory.class

```java
@Component
@FeignClient(value = "springcloud-provider-dept",fallbackFactory = DeptClientServiceFallBackFactory.class)
public interface DeptClientService {

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);

    @GetMapping("/dept/get/{id}")
    Dept get(@PathVariable("id") int id);

    @PostMapping("/dept/getAll")
    List<Dept> getAll();
}
```

修改springcloud-consumer-dept-feign-80的application.yml配置文件，添加

```yml
# 开启服务降级
feign:
  hystrix:
    enabled: true
```

### 3.3 体验

启动7001Eureka中心、8001服务提供者、80客户端

正常情况下，请求是正常的

![image-20210209164247491](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209164247491.png)

关闭8001服务提供者后，再次请求

![image-20210209164211824](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209164211824.png

## 4 服务熔断vs服务降级

* 服务熔断：
	* 在==服务端==做
	* 是调用提供者服务在出现异常或超时的一种处理方案

* 服务降级：
	* 在==客户端==做
	* 提供者只管关闭服务，客户端自己来处理错误
	* 即当某个服务熔断或关闭之后，服务不再被调用，此时在客户端自己准备一个FallBackFactory，返回一个缺省值

所以从上述分析来看，两者其实从有些角度看是有一定的类似性的：

1. **目的很一致**，都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
2. **最终表现类似**，对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
3. **粒度一般都是服务级别**，当然，业界也有不少更细粒度的做法，比如做到数据持久层（允许查询，不允许增删改）；
4. **自治性要求很高**，熔断模式一般都是服务基于策略的自动触发，降级虽说可人工干预，但在微服务架构下，完全靠人显然不可能，开关预置、配置中心都是必要手段；

而两者的区别也是明显的：

1. **触发原因不太一样**，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
2. **管理目标的层次不太一样**，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
3. **实现方式不太一样**，这个区别后面会单独来说；



## 5 原理

1. 用一个HystrixCommand 或者 HystrixObservableCommand （这是命令模式的一个例子）包装所有的对外部系统（或者依赖）的调用，典型地它们在一个单独的线程中执行
2. 调用超时时间比你自己定义的阈值要长。有一个默认值，对于大多数的依赖项你是可以自定义超时时间的。
3. 为每个依赖项维护一个小的线程池(或信号量)；如果线程池满了，那么该依赖性将会立即拒绝请求，而不是排队。
4. 调用的结果有这么几种：成功、失败（客户端抛出异常）、超时、拒绝。
5. 在一段时间内，如果服务的错误百分比超过了一个阈值，就会触发一个断路器来停止对特定服务的所有请求，无论是手动的还是自动的。
6. 当请求失败、被拒绝、超时或短路时，执行回退逻辑。
7. 近实时监控指标和配置变化。

当你使用Hystrix来包装每个依赖项时，上图中所示的架构会发生变化，如下图所示：

每个依赖项相互隔离，当延迟发生时，它会被限制在资源中，并包含回退逻辑，该逻辑决定在依赖项中发生任何类型的故障时应作出何种响应：

![image-20210208174234418](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210208174234418.png)

## 参考

[Hystrix的服务熔断和服务降级](https://www.cnblogs.com/guanyuehao0107/p/11848286.html)

[Hystrix官网](https://github.com/Netflix/Hystrix/wiki)

[Hystrix介绍](https://www.cnblogs.com/cjsblog/p/9391819.html)

[视频：狂神说Spring Cloud](https://www.bilibili.com/video/BV1jJ411S7xr)

[狂神说SpringCloud学习笔记](https://blog.csdn.net/weixin_43591980/article/details/106255122#t1)