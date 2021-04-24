[TOC]

## 课前复习

### DI的三种方式

1. 自动化配置

	* `@Component` + `@Autowired`

2. JavaConfig

	* 在`@Configuration`类下使用`@Component`注解返回一个bean、

	* 注入

		✓ 调用方法（） 

		✓ 通过方法参数自动装配（其它配置类、其它方式创建的Bean）

3. XML配置

	* `<bean>`和`<property>`标签

	![image-20210424100123796](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210424100123796.png)

### AOP术语

* 通知（**Advice**）：

	定义了切入的逻辑，切面做什么以及何时做，之前、之后、异常？

* 切点（**Pointcut**）：何处

	用切点表达式来表达，定义在哪些地方做切入

* 切面（**Aspect**）：Advice和Poincut的结合 

* 连接点（Join point） 

* 引入（introduction）：为现有的类引入新的行为和状态 

* 织入（Weaving）：切面应用到目标对象的过程

### 简述spring security提供的基本能力有哪些？

* 从web层面

	对每一个用户进行认证

	对每一个URL进行授权

	* 有怎样的角色，才能访问？

* 从方法层面，可以从方法上看用户是否有某些权限

* 启动配置

	```java
	@Configuration
	@EnableWebMvcSecurity
	public class SecurityConfig extends WebSecurityConfigurerAdapter {
	    // 配如何通过拦截器保护请求
	    void configure(HttpSecurity http)
	    // 配用户数据存储
	    void configure(AuthenticationManagerBuilder auth)
	    	// 内存、数据库表
	}
	```

	

* 可配置的项目

	![image-20210318193456855](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210318193456855.png)

	用户数据存储方式：内存、数据库

* 安全注解

	在需要授权的方法前加Secured，在角色前面加ROLE变成权限

	![image-20210318193711584](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210318193711584.png)

### spring提供的配置数据源bean的方式有哪些

1. 连接池
2. jdbc
3. 配置嵌入式数据源
4. JNDI

### 简述Spring提供的持久化方案jdbc template、JPA、hibernate的开发过程、特征和异同点

共同特点：需要定义数据源（四种数据源？

* jdbc template：模板方法，只需要提供标准select语句即可，有参数的话可以使用占位符

* hibernate：定义对象和表之间的关系，注解@Entity、@Column（javax.persistence

	**调用方法前**需要获取session对象，需要定义一个sessionFactory，然后才能获取session

	三类查询：

	* HQL
	* QBC
	* 本地SQL查询

	问答题、选择题

* JPA：需要有EntityManager

	接口继承JpaRepository或CRUDRepository

	* 查询方法使用领域特定语言  查询动词：get、read、find、count

	* 自定义：@Query

	* 加@Repository注解

	* 完全自己实现，命名为接口+Impl，使用EntityManager直接底层实现

		```java
		  @PersistenceContext
		  private EntityManager entityManager;
		```


### 简述缓存开发过程

1. 初始化缓存管理器，cacheManeger

	三种，concurrentMap、EhCache、Redis

	EhCache支持数据持久化 √

2. 加@EnableCaching注解

3. 在需要缓存的方法上加注解

	* @Cacheable 判断有没有，有的话做缓存，并保存
	* @CachePut 不会去缓存中拿，会调用完放到缓存
	* @CacheEvict 

### REST

资源：网络上的一个实体，标识：URI

HTTP协议的四个操作方式的动词：GET、POST、PUT、DELETE 

* CRUD：Create、Read、Update、Delete

### 微服务开发相关注解

* `@SpringBootApplication`
	* ✓ 配置类`@Configuration `
	* ✓ `@ComponentScan `
* `@RestController `
	* ✓ `@Controller `
	* ✓ 请求响应，JSON编解码（序列化）

### 配置不同层级

1. 配置信息硬编码到代码中

2. 分离的外部属性文件
3. 与物理部署分离，如外部数据库
4. 配置作为单独的服务提供（配置管理服务）
5. 配置管理更改需要通知到使用数据的服务

### 配置服务使用的存储库类型

共享文件系统

源代码控制下的文件（Git）

关系数据库

nosql数据库

Spring Cloud Config ：文件系统、Git、Eureka、Consul

### 简述spring cloud在分布式系统中配置服务开发和使用

即如何开发配置服务？

* 服务端

	* 依赖

		spring-cloud-config-server

		spring-cloud-starter-config

	* 使用注解开启服务@EnableConfigServer

	* application.yml中指定数据源（git、本地

* 客户端

	* 加相关依赖

		spring-cloud-starter-config

	* 指定到哪去获取配置服务？

		spring.cloud.config.url属性

		可以在bootstrap.yml或-d参数指定，指定服务名configserver

		根据spring.application.name和spring.profile.active去找配置文件

	* 通过@Value注解获取数据

		如要需要在运行时能够刷新，需要在方法或者类上使用@RefreshScope注解

### 简述配置服务在分布式服务中的作用

* 通过配置服务统一提供数据（服务数量多，如果使用传统的方式，维护起来不方便）
* 云平台上方便使用多种数据源，比如git、KV数据库等

### 服务调用的方式

1. ==SpringDiscoveryClient==

	spring-cloud-starter-ribbon

	启动类加@EnableDiscoveryClient，使能够使用 DiscoveryClient和Ribbon库 

	注入：private DiscoveryClient discoveryClient; 

	discoveryClient.getInstances 

	new RestTemplate 

	restTemplate.exchange

2. ==支持Ribbon的RestTemplete==

	spring-cloud-starter-ribbon

	注入带@LoadBalanced的restTemplate

	restTemplate方法，通过服务名进行调用

3. ==Feign==

	依赖：spring-cloud-starter-feign

	注解：@EnableFeignClients

	定义接口并加注解：@FeignClient("服务名")

### 什么是弹性？

客户端弹性，在客户端我们可以做什么？

远程资源不可靠，可能会尝试超时，对于远程访问不可用的情况下，客户端可以采取四个措施：

* Ribbon：每隔30s从eureka server获取服务
* fallback：服务不可用时走其他路径，可能访问不同的服务
* 舱壁隔离模式：hystrix 默认共用一个线程池，如果某一个服务不可用，线程池会被占满。每个访问服务的地方单独定义一个线程池，以此不影响其他服务
* 断路器模式：一段时间进行一次统计，失败次数达到阈值，会进行熔断，报错或者进入后备模式；有一个sleep时间，时间到了以后进行重试 

![image-20210422193133473](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422193133473.png)

## 选择题

1. **spring可以支持第三方框架** √

2. **横切关注点包括**？

	日志、安全、事务、缓存

3. **通知：advice 有哪些注解**？

	```java
	@Before
	@After
	@AfterReturning
	@AfterThrowing
	@Around
	```

4. **AOP生效要加哪些注解：**

	```java
	@Configuration
	@EnableAspectJAutoProxy // 开启AspectJ的自动代理机制，配置类中
	public class ConcertConfig {
	    @Bean
	    public Audience2 audience() { //定义Audience的bean
	        return new Audience2();
	    }
	}
	
	@Aspect
	public class Audience1 {
	    @Pointcut("execution(* concert.Performance.perform( .. ))")
	    public void performance() {
	    }
	
	    @Before("performance()")
	    // 等价于 @Before("execution(* concert.Performance.perform( .. ))")
	    public void silenceCellPhones() {
	        System.out.println("Silencing cell phones");
	    }
	    
	    ...
	}
	```

5. **有component能力的注解** 

	@Controller @Repository @Service

6. **Aspect指什么？**

	Advice和Poincut的结合 

7. **实现控制器 @RequestMapping 是不是只能加在方法上？**

	不是 还可以加在类上

8. **启动SpringMVC框架使用什么注解**

	@EnableWebMvc

	@Controller/RestController

	@RequestMapping("")

	@Service

	@Repository

	....

9. **控制器返回类型**

	* ModelAndView，并对Model和View分别进行设置

	* 返回一个对象，作json转换

	* 字符串（代表逻辑视图名、redirect重定向、forward转发）

	* 返回void，在Controller方法的形参中定义HTTPServletRequest和HTTPServletResponse对象进行请求的接收和响应

10. **数据源，DriverManagerDataSource有没有做池化处理？**

	没有

	SingleConnectionDataSource:只有一个连接的池

11. **生产环境中：配置信息放到配置服务中**

	* 通过配置服务统一提供数据
	* 云平台上方便使用多种数据源

12. **业务层和持久层用接口作隔离，好处是什么？**

	1. 方便测试 √
	2. 方便更换数据源 × 没关系 
	3. 方便更换持久化实现层 √
	4. 屏蔽底层数据访问异常 × 没有，转成runtime异常，可以抓

13. **Hibernate访问数据库** 

	1. 拿到session
	2. 定义映射关系（Java代码中注解定义和数据库表的关系）
	3. HQL、QBC、支持SQL查询

14. **JPA，不需要定义数据对象和数据表之间的映射关系** 

	×

15. **Spring data jpa 如何开发？**

	接口继承JpaRepository或CRUDRepository

	* 查询方法使用领域特定语言  查询动词：get、read、find、count
	* 自定义：@Query
	* 加@Repository注解
	* 完全自己实现，命名为接口+Impl，使用EntityManager直接底层实现

	```java
	  @PersistenceContext
	  private EntityManager entityManager;
	```

16. **mongodb访问**

	* spring data mongodb支持，继承MongoRepository，可以获得默认的查询，也可以自己定义
	* @Document、@Field（Spring提供，区别于@Entity是java提供的）
	* 存数据的时候，Java对象不需要序列化
	* 了解一些概念，connection，和关系型数据库的对应关系，shell可以写js代码  ![image-20210424112925111](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210424112925111.png)

17. **redis的特点，支持的数据类型**

	* 需要指定序列化方式，不指定的话默认是使用JDK的序列化方式，需要implements Serializable
	* KV，Hash结构
	* 内存数据库（用于缓存）
	* 支持集群
	* 主从复制
	* 可以数据持久化
	* KV**区分大小写**
	* 数据类型：string（字符串）hash（哈希）list（列表）set（集合）zset(sorted set：有序集合)

18. **缓存**

	* EhCache和Redis的区别？
		* ehcache直接在jvm虚拟机中缓存，速度快，效率高；但是缓存共享麻烦，集群分布式应用不方便。
		* redis是通过socket访问到缓存服务，效率比ecache低，比数据库要快很多

	* EhCache支持数据持久化吗？支持

	* 常用的注解：@EnableCaching、@Cacheable、@CachePut、@CacheEvict

19. **容器**、

	* 容器和虚拟机的区别？容器更轻量级，启动更快
	* 容器镜像是分层的，只有最上面一层可读写，其他可读  √
	* 虚拟机是操作系统级别的资源隔离，容器本质上是进程级的资源隔离 √

20. **docker run的命令参数**

	--rm 退出的时候会自动删除

	-P 随机端口映射

	 docker port [容器]  查看端口映射

	--name 给容器起名字

	-d 后台运行 

	-e 传递环境变量

	哪四个管理类命令

	* docker images 管理镜像
	* docker container 管理容器
	* docker network 管理网络
	* docker volume 管理容器卷

	在容器里看IP地址

	* `cat /etc/hosts`

	如果一个容器挂载到两个网络里，分别有一个IP地址

21. **SpringBoot和SpringCloud的关系**

	Spring Boot提供了基于java的、面向REST的微服务框架 

	Spring Cloud对各种第三方库做集成，使实施和部署微服务到私有云或公有云变得更加简单

22. **Spring-cloud能解决的问题**

	* 微服务划分，服务粒度、通信协议、接口设计、配置管理、使用事件解耦微服务

	* 服务注册、发现和路由

		*去哪里找服务的IP、端口号？*

	* 弹性，负载均衡，断路器模式（熔断），容错

		*分布式系统当中可能的出错，如服务不存在了、网络出现错误了等的解决方法*

	* 可伸缩

		*服务负载增加，服务端可以动态增加服务示例*

	* 日志记录和跟踪

		*日志系统要横跨多个微服务时，做一个聚集*

	* 安全

		*分布式系统中用户的认证和授权问题*

	* 构建和部署，基础设施即代码

		*方便部署，从一个干净的环境中安装所需要的依赖*

23. **当一个服务需要向配置服务获取配置数据时候，配置服务怎么知道要给哪些数据？**

	通过==服务名和profile==

	分布式系统中有很多微服务，怎么从配置服务中找自己所需要的数据？？

	* 每个微服务有一个属性spring.application.name，就根据这个name去找

		AKA appid

	* 会去找appid开头的yml文件

	![image-20210415194824747](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210415194824747.png)

24. **实现配置服务，加什么注解？**

	@EnableConfigServer

25. **使用配置服务，加什么注解？**

	什么都不需要

26. **在服务网关处可以实现怎样的能力？**

	用户认证和授权，静态路由、动态路由，数据收集，日志

## 简答题

### web开发处理请求过程

![image-20210311201424892](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210311201424892.png)

1. 请求过来，第一站到达DispatcherServlet

	不一定所有的请求都到DispatcherServlet，比如静态页面，系统有好多Servlet；如果让Spring接管，请求都要到DispatcherServlet

2. HandleMapping维护了对应关系，通过HandleMapping，找到对应的Controller

3. 将请求转到controller，请求处理，将参数转为合适的Java对象，然后找到Service进行处理（Service可能会做数据的持久化等），并将结果返回给controller

4. 返回model和view

5. 根据view找到视图解析器，把model传过去

6. 视图解析器根据model渲染页面

7. 返回给前端

注：3后可能也有其他路径，比如前后端分离下，直接把结果转成json字符串返回

> DispatcherServlet是核心
>
> Controller是处理请求的核心

**考试：根据这个图解释一下一个请求的处理过程**

### web层次结构

![image-20210424095212214](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210424095212214.png)

### 微服务相比传统单体有何优势，单体有何不足？

微服务的特征有：

1. 将应用程序分解为具有明确定义了职责范围的细粒度组件 
2. 完全独立部署，独立测试，并可复用 
3. 使用轻量级通信协议，HTTP和JSON，松耦合 
4. 服务实现可使用多种编程语言和技术 
5. 将大型团队划分成多个小型开发团队，每个团队只负责他们各自的服务

单体应用程序的不足之处在于

1. 数据库表对所有模块可见
2. 一个人的修改整个应用都要重新构建、测试、部署
3. 整体复制分布式部署，不能拆分按需部署

### 服务注册与发现的好处

1. **快速的水平伸缩，而不是垂直伸缩**

	*什么是水平伸缩、垂直伸缩？水平伸缩也叫横向伸缩，当处理能力不够的时候，请求数越来越多，可以横向扩充机器（如增加容器）*

	*客户端只需要提供服务名字就可以了，横向伸缩对客户端来说是透明的*

2. **提高应用程序的弹性**

	弹性：指系统出错以后系统对错误处理的能力

	当系统中的服务突然挂了或者新增加，注册服务可以发现

	两个重要工具：EurekaClient、Ribbon，可以做到负载均衡，向服务代理进行最新状态查询

### Eureka Ribbon Zuul相互关系

eurekaclient ribbon feign zuul如何配合？

* Eureka

	服务注册中心

	依赖：spring-cloud-starter-eureka-server

	注解：@EnableEurekaServer

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

	需要注册的服务，只需要导入spring-cloud-starter-eureka依赖，并加@EnableEurekaClient注解

	并配置：

	```yml
	# eureka配置，服务注册到哪里
	eureka:
	  client:
	    service-url:
	      defaultZone: http://localhost:7001/eureka
	  instance:
	    instance-id: spring-cloud-provider-dept8001 # 修改eureka上的描述信息
	```

* Ribbon

	提供客户端的软件负载均衡算法

	依赖：spring-cloud-starter-ribbon

	注解：@EnableEurekaClient

	注入restTemplate

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

	调用restTemplate方法，指定要调用的服务名，而不是IP

* Feign

	依赖：spring-cloud-starter-feign

	注解：@EnableFeignClients

	定义接口并加注解：@FeignClient("服务名")

	```java
	@Component
	@FeignClient(value = "springcloud-provider-dept")
	public interface DeptClientService {
	    @PostMapping("/dept/add")
	    boolean addDept(Dept dept);
	}
	```

* Zuul

	spring-cloud-starter-zuul

	@EnbaleZuulProxy

	网关，可以做用户认证和授权，静态路由、动态路由，数据收集，日志等

	Zuul服务内部可以带Ribbon进行负载均衡，对外部请求做负载均衡

> Zuul是总的入口
>
> Zuul需要向eureka去获取服务和服务的状态，并建立相应的路由
>
> 由zuul来访问
>
> 借助ribbon来访问目标服务，负载均衡
>
> Feign简化了访问目标服务的方式，实现接口就行
>
> 背后还是要借助于ribbon