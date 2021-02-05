# SpringCloud(四)：服务发现与Ribbon负载均衡详解

[TOC]



## 1 负载均衡及Ribbon

### 1.1 Ribbon是什么

- Spring Cloud Ribbon 是基于Netflix Ribbon 实现的一套**客户端负载均衡的工具**。
- 简单的说，Ribbon 是 Netflix 发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将 Netflix 的中间层服务连接在一起。Ribbon 的客户端组件提供一系列完整的配置项，如：连接超时、重试等。简单的说，就是在配置文件中列出 LoadBalancer (简称LB：负载均衡) 后面所有的及其，Ribbon 会自动的帮助你基于某种规则 (如简单轮询，随机连接等等) 去连接这些机器。我们也容易使用 Ribbon 实现自定义的负载均衡算法！

> 关键词：客户端、负载均衡

### 1.2 负载均衡

- LB，即负载均衡 (LoadBalancer) ，在微服务或分布式集群中经常用的一种应用。
- 负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA (高用)。
- 常见的负载均衡软件有 Nginx、Lvs 等等。
- Dubbo、SpringCloud 中均给我们提供了负载均衡，**SpringCloud 的负载均衡算法可以自定义**。
- 负载均衡简单分类：
	- 集中式LB
		- 即在服务的提供方和消费方之间使用独立的LB设施，如**Nginx(反向代理服务器)**，由该设施负责把访问请求通过某种策略转发至服务的提供方！
	- 进程式 LB
		- 将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器。
		- **Ribbon 就属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址！

![image-20210205163122614](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205163122614.png)

> 关键词：正向代理

## 2 SpringBoot集成Ribbon

> * 在springcloud-consumer-dept-80模块中修改，[Rest环境搭建](https://www.cnblogs.com/cpaulyz/p/14364353.html)
> * 需要构建eureka集群，详情查看[构建集群Eureka Server#](https://www.cnblogs.com/cpaulyz/p/14372252.html#4-拓展：构建集群eureka-server)

### 2.1 集成Ribbon

> 将Ribbon集成到SpringBoot中，需要依赖Eureka

导入依赖：ribbon和eureka

```xml
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
```

在配置文件application.yml中添加

```yml
#  eureka
eureka:
  client:
    register-with-eureka: false # 不注册自己
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

在启动类上添加上Eureka注解

```java
@EnableEurekaClient
@SpringBootApplication
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class,args);
    }
}
```

### 2.2 服务发现

> 通过Eureka进行服务发现

修改RestTemplate配置，加上`@LoadBalanced`注解

```java
@Configuration
public class BeanConfig {
    @Bean
    @LoadBalanced // 配置负载均衡实现restTemplate
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

回顾一下之前的客户端，我们之间把请求URL写死了

```java
@RestController
@RequestMapping("/consumer/dept")
public class DeptConsumerController {
    // 理解：消费者，不应该有service层
    // RestTemplate 供我们直接调用
    // 注册到Spring中
    @Autowired
    RestTemplate restTemplate;

    private static final String REST_URL_PREFIX = "http://localhost:8001";

    @RequestMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
    }

    @RequestMapping("/add")
    public boolean add(Dept dept){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
    }

    @RequestMapping("/getAll")
    public List<Dept> getAll(){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/getAll",null,List.class);
    }

}
```

为了实现负载均衡，我们肯定得从eureka中去动态获取请求地址，想法就是通过名字！

![image-20210205165609893](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205165609893.png)

很简单，这里只需要修改请求为`REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";`即可

修改后如下

```java
@RestController
@RequestMapping("/consumer/dept")
public class DeptConsumerController {
    // 理解：消费者，不应该有service层
    // RestTemplate 供我们直接调用
    // 注册到Spring中
    @Autowired
    RestTemplate restTemplate;

    // Ribbon: 这里地址应该是一个变量，通过服务名来访问
//    private static final String REST_URL_PREFIX = "http://localhost:8001";
    private static final String REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";

    @RequestMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
    }

    @RequestMapping("/add")
    public boolean add(Dept dept){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
    }

    @RequestMapping("/getAll")
    public List<Dept> getAll(){
        return restTemplate.postForObject(REST_URL_PREFIX+"/dept/getAll",null,List.class);
    }
}
```

![image-20210205171610260](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205171610260.png)

## 3 使用Ribbon实现负载均衡

在上面，我们实现了通过Ribbon和Eureka来进行服务发现，这里我们要进行真正的负载均衡

### 3.1 构建分布式的服务

这之前，我们的服务只有一个实例`springcloud-provider-dept8001`，这里我们要另外做两个实例springcloud-provider-dept8002和springcloud-provider-dept8003

创建方法类似[springcloud-provider-dept8001#](https://www.cnblogs.com/cpaulyz/p/14364353.html#4-springcloud-provider-dept-8001)，这里只列出了几个需要注意的要点

* server.port端口不同
* spring.application.name相同，都是 springcloud-provider-dept
* eureka.instance.instance-id不同
* 启动类不同
* 连接数据库不同，其中8001连接springcloud，8002连接springcloud02，8003连接springcloud03
* 数据库中的dbSource字段不同

创建完成后，数据库应该是这样的：

![image-20210205174130169](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205174130169.png)

项目结构是这样的：

![image-20210205175052028](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205175052028.png)

完成构建后，访问eureka中心，可以看到类似画面：

![image-20210205175307757](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205175307757.png)

简单测试一下，==dbSource是我们区分服务的重要依据！==

![image-20210205175551762](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205175551762.png)

### 3.2 实现负载均衡

如果你已经构建好了上面的分布式服务，并尝试通过配置了Ribbon消费者进行请求，就会发现我们已经实现了负载均衡了！

| 第一次访问                                                   | 第二次访问                                                   | 第三次访问                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image-20210205175950684](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205175950684.png) | ![image-20210205180037951](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205180037951.png) | ![image-20210205180021877](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205180021877.png) |

原理很简单

1. 服务提供者向Eureka注册中心进行注册，spring.application.name相同而eureka.instance.instance-id不同，从而该服务有三个示例。
2. 消费者通过Eureka注册中心使用服务，Ribbon进行负载均衡，默认使用的是**轮询**的策略，进而在客户端实现负载均衡

![image-20210205180108886](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205180108886.png)

## 4 自定义负载均衡策略

> Ribbon默认使用的是轮询的策略，我们可以自定义负载均衡策略

### 4.1 相关接口

相关接口：IRule

```java
public interface IRule {
    Server choose(Object var1);
    void setLoadBalancer(ILoadBalancer var1);
    ILoadBalancer getLoadBalancer();
}
```

有很多实现类

![image-20210205180537886](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205180537886.png)

* AvailabilityFilteringRule ： 会先过滤掉，跳闸，访问故障的服务，对剩下的进行轮询    
* RetryRule ： 会先按照轮询获取服务，如果服务获取失败，则会在指定的时间内进行，重试
* RoundRobinRule：轮询

* ...

### 4.2 理解源码

这里可以看一下轮询的源码，其实很好理解，就是维护一个计数器

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {
    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;
    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        this.nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        this.setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else {
            Server server = null;
            int count = 0;

            while(true) {
                if (server == null && count++ < 10) {
                    List<Server> reachableServers = lb.getReachableServers();
                    List<Server> allServers = lb.getAllServers();
                    int upCount = reachableServers.size();
                    int serverCount = allServers.size();
                    if (upCount != 0 && serverCount != 0) {
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);
                        server = (Server)allServers.get(nextServerIndex);
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }

                            server = null;
                        }
                        continue;
                    }

                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }

                if (count >= 10) {
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }

                return server;
            }
        }
    }

    private int incrementAndGetModulo(int modulo) {
        int current;
        int next;
        do {
            current = this.nextServerCyclicCounter.get();
            next = (current + 1) % modulo;
        } while(!this.nextServerCyclicCounter.compareAndSet(current, next));

        return next;
    }

    public Server choose(Object key) {
        return this.choose(this.getLoadBalancer(), key);
    }

    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

如果需要修改的话，只需要修改核心的`choose`方法即可！

### 4.3 自定义

模仿源码，自己写一个`RoundThreeRule`，规则是

* 轮询，每个实例访问三次

其实只需要修改choose()方法中调用的`incrementAndGetModulo`即可，下面是修改过的代码

```java
public class RoundThreeRule extends AbstractLoadBalancerRule {
    private AtomicInteger nextServerCyclicCounter;
    private AtomicInteger cnt;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;
    private static Logger log = LoggerFactory.getLogger(RoundThreeRule.class);

    public RoundThreeRule() {
        this.nextServerCyclicCounter = new AtomicInteger(0);
        this.cnt = new AtomicInteger(0);
    }

    public RoundThreeRule(ILoadBalancer lb) {
        this();
        this.setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else {
            Server server = null;
            int count = 0;

            while(true) {
                if (server == null && count++ < 10) {
                    List<Server> reachableServers = lb.getReachableServers();
                    List<Server> allServers = lb.getAllServers();
                    int upCount = reachableServers.size();
                    int serverCount = allServers.size();
                    if (upCount != 0 && serverCount != 0) {
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);
                        server = (Server)allServers.get(nextServerIndex);
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }

                            server = null;
                        }
                        continue;
                    }

                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }

                if (count >= 10) {
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }

                return server;
            }
        }
    }

    // 修改过的方法
    private int incrementAndGetModulo(int modulo) {
        int currentCnt = cnt.incrementAndGet();
        if(currentCnt>=3){
            int tmp = 0;
            do {
                currentCnt = cnt.get();
            } while(!this.cnt.compareAndSet(currentCnt, tmp));
            int current;
            int next;
            do {
                current = this.nextServerCyclicCounter.get();
                next = (current + 1) % modulo;
            } while(!this.nextServerCyclicCounter.compareAndSet(current, next));
            return next;
        }else{
            return this.nextServerCyclicCounter.get();
        }

    }

    public Server choose(Object key) {
        return this.choose(this.getLoadBalancer(), key);
    }

    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

之后进行配置

```java
    @Bean
    public IRule myRule(){
        return new RoundThreeRule();
    }
```

即可

## 5 更优雅的Ribbon配置方式:@RibonClient

> 我们在上文进行了全局的IRule替换，但在实际开发中，我们可能会遇到这种情况：对于不同的服务，我们需要配置不同的负载均衡策略，使用Ribbon怎样实现？
>
> ——这就需要更加优雅的Ribbon配置方式：`@RibonClient`

1. 另起一个配置类

	```java
	@Configuration
	public class RibbonRoundThreeConfig {
	    @Bean
	    public IRule myRule(){
	        return new RoundThreeRule();
	    }
	}
	```

	==注意：该类不应该被应用程序上下文的@ComponentScan注解扫描到==，可以放到启动类的同级目录，比如

	![image-20210205190315619](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210205190315619.png)

2. 在SpringBoot配置类中，进行@RibonClient配置

	```java
	@EnableEurekaClient
	@SpringBootApplication
	@RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT",configuration = RibbonRoundThreeConfig.class)
	public class DeptConsumer_80 {
	    public static void main(String[] args) {
	        SpringApplication.run(DeptConsumer_80.class,args);
	    }
	}
	```

	* `name`是服务名，对应eureka中微服务的名字
	* `configuration`指明配置文件，比如这里定义了负载均衡策略

### 5.1 @LoadBalanced和@RibbonClient的区别

#### (1)`@LoadBalanced`

用作标记注释，指示被注释的对象`RestTemplate`应使用`RibbonLoadBalancerClient`与您的服务进行交互。

反过来，这允许您对传递给的网址使用“逻辑标识符” `RestTemplate`。这些逻辑标识符通常是服务的名称。例如：

```
restTemplate.getForObject("http://some-service-name/user/{id}", String.class, 1);
```

`some-service-name`逻辑标识符在哪里。

#### (2)`@RibbonClient`

用于配置功能区客户端。

**是否需要@RibbonClient？**

没有！如果您正在使用服务发现，并且对所有默认的功能区设置都没问题，那么甚至不需要使用`@RibbonClient`注释。

**我`@RibbonClient`什么时候应该使用？**

至少有两种情况需要使用 `@RibbonClient`

1. 您需要为特定的功能区客户端自定义功能区设置
2. 您没有使用任何服务发现

***自定义功能区设置：\***

定义一个 `@RibbonClient`

```
@RibbonClient(name = "some-service", configuration = SomeServiceConfig.class)
```

- `name` -将其设置为与功能区调用的服务相同的名称，但需要其他自定义功能区以与功能区交互。
- `configuration`-将其设置为`@Configuration`所有定义为的类`@Beans`。确保 **不** 选择此类，`@ComponentScan`否则它将覆盖所有功能区客户端的默认设置。

请参阅Spring Cloud
Netflix文档中的“自定义RibbonClient”部分[（链接）](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#customizing-the-ribbon-client)

***在不进行服务发现的情况下使用功能区\***

如果您未使用Service
Discovery，则注释`name`字段`@RibbonClient`将用于`application.properties`在您传递给的URL
中的前缀以及“逻辑标识符”中为您的配置添加前缀`RestTemplate`。

定义一个 `@RibbonClient`

```
@RibbonClient(name = "myservice")
```

然后在你的 `application.properties`

```
myservice.ribbon.eureka.enabled=false
myservice.ribbon.listOfServers=http://localhost:5000, http://localhost:5001
```

## 参考

[@RibbonClient和@LoadBalanced之间的区别](http://codingdict.com/questions/36297)

[为Ribbon Client自定义配置](https://blog.csdn.net/m0_37592952/article/details/89975057)

[视频：狂神说Spring Cloud](https://www.bilibili.com/video/BV1jJ411S7xr)

[狂神说SpringCloud学习笔记](https://blog.csdn.net/weixin_43591980/article/details/106255122#t1)

