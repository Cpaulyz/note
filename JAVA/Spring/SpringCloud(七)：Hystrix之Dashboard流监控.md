# SpringCloud(七)：Hystrix之Dashboard流监控

[TOC]

> Hystrix Dashboard，它主要用来实时监控Hystrix的各项指标信息。通过Hystrix Dashboard反馈的实时信息，可以帮助我们快速发现系统中存在的问题。下面通过一个例子来学习

## 1 Demo

### 1.1 编写dashboard模块

新建module，springcloud-consumer-hystrix-dashboard

导入依赖

```xml
    <dependencies>
        <!--hystrix依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <!--hystrix监控-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
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
```

配置文件application.yml

编写启动类

```java
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumerDashboard_9001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashboard_9001.class,args);
    }
}
```

启动后，访问`localhost:xxxx/hystrix`可以看到以下页面

![image-20210209171738255](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209171738255.png)



### 1.2 修改服务提供者

这里以springcloud-provider-dept-hystrix-8001为例

保证服务提供者有以下依赖

```xml
    	 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>        
		<!--actuator监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

在主启动类中添加一个servlet

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient // 服务发现
@EnableCircuitBreaker // 添加对熔断的支持
public class DeptProviderHystrix_8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8001.class,args);
    }

    //增加一个 Servlet
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        //访问该页面就是监控页面
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}
```

### 1.3 体验

启动7001Eureka中心，8001服务提供者，9001监控，80客户端

在`localhost:9001/hystrix`中进行注册

![image-20210209172137689](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209172137689.png)

不断使用80客户端发请求，即可看到监控页面发生变化

![image-20210209172307832](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209172307832.png)

## 2 Dashboard详解

 在监控的界面有两个重要的图形信息：一个实心圆和一条曲线。

* 实心圆：
	1. 通过颜色的变化代表了实例的健康程度，健康程度从绿色、黄色、橙色、红色递减。
	2. 通过大小表示请求流量发生变化，流量越大该实心圆就越大。所以可以在大量的实例中快速发现故障实例和高压实例。

* 曲线：
	* 用来记录2分钟内流浪的相对变化，可以通过它来观察流量的上升和下降趋势。

![image-20210209172335197](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210209172335197.png)

## 参考

[Hystrix-Dashboard仪表盘](https://www.cnblogs.com/happyflyingpig/p/8372485.html)

[Hystrix官网](https://github.com/Netflix/Hystrix/wiki)

[Hystrix介绍](https://www.cnblogs.com/cjsblog/p/9391819.html)