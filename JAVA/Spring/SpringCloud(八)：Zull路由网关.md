# SpringCloud(八)：Zuul路由网关

[TOC]



## 1 介绍

### 1.1 什么是Zuul

> [Zuul官方介绍](https://github.com/Netflix/zuul/wiki)

Zuul包含了对请求的**路由**(用来跳转的)和**过滤**两个最主要功能：

其中路由功能负责将外部请求转发到具体的微服务实例上，是实现**外部访问统一入口**的基础，而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础。

Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

* **注意**：Zuul 服务最终还是会注册进 Eureka
* **提供**：代理 + 路由 + 过滤 三大功能！

![image-20210210172030566](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210172030566.png)

回顾一下Spring的框架图，路由网关是在最上层的一个应用

![image-20210202164104251](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202164104251.png)

## 2 demo

> 完整项目结构（含3中的实战）
>
> ![image-20210210181457800](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210181457800.png)

### 2.1 构建module

创建一个新的module，名为springcloud-zuul-9527

依赖

```xml

    <dependencies>
        <!--zuul-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
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

```yml
server:
  port: 9527

#  spring配置
spring:
  application:
    name: springcloud-zuul-gateway

# eureka配置，服务注册到哪里
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
#      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: spring-cloud-zuul # 修改eureka上的描述信息

# info配置
info:
  app.name: cpaulyz-springcloud
  company.name: cpaulyz
```

启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication_9527 {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication_9527.class,args);
    }
}
```

### 2.2 体验

启动Eureka7001、Zuul9527、服务提供者8001

在Eureka界面应该可以看到

![image-20210210174112504](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210174112504.png)

接下来体验一下通过路由网关来访问服务

原本访问服务的方式是这样的：

![image-20210210174154064](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210174154064.png)

通过路由网关，我们可以从`路由网关/服务名/路径`进行访问

![image-20210210174253759](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210174253759.png)

### 2.3 更详细的配置

我们已经成功通过Zuul来访问服务了，但是还存在着一些问题

* 需要暴露出微服务名字，可否隐藏？

因此，需要通过更详细的配置来实现！

```yaml
# zuul 路由网关配置
zuul:
  # 路由相关配置
  # 原来访问路由 eg:http://www.cpaulyz.com:9527/springcloud-provider-dept/dept/get/1
  # zuul路由配置后访问路由 eg:http://www.cpaulyz.com:9527/cpaulyz/mydept/dept/get/1
  routes:
    mydept.serviceId: springcloud-provider-dept # eureka注册中心的服务提供方路由名称
    mydept.path: /mydept/** # 将eureka注册中心的服务提供方路由名称 改为自定义路由名称
  # 不能再使用这个路径访问了，*： 忽略,隐藏全部的服务名称~
  ignored-services: "*"
  # 设置公共的前缀
  prefix: /cpaulyz
```

重启项目后，新的访问方式：

![image-20210210174640312](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210174640312.png)

原来的访问方式：

![image-20210210174820749](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210174820749.png)

## 3 实战：实现登录认证

> 我们利用Zuul来实现一个登陆认证，在请求头部中带有token判断为有效

创建Filter类，实现ZuulFilter，并添加`@Component`注解，注册到Spring容器中

```java
@Component
public class LoginFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 过滤器顺序，越小越先执行
     *
     * @return
     */
    @Override
    public int filterOrder() {
        return 4;
    }

    /**
     * 是否生效
     * @return
     */
    @Override
    public boolean shouldFilter() {
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();
        String requestURI = request.getRequestURI();
        // 根据URI判断是否需要过滤...
        return true;
    }

    /**
     * 业务逻辑
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        //JWT
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        // token对象
        String token = request.getHeader("token");
        //登录校验逻辑  根据公司情况自定义 JWT
        if (StringUtils.isBlank(token)) {
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```

然后重启项目

不带token访问：

![image-20210210181212419](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210181212419.png)

带token访问：

![image-20210210181227761](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210210181227761.png)

## 参考

[SpringCloud zuul网关以及实现登录验证](https://blog.csdn.net/xcc_2269861428/article/details/109073830)

[SpringCloud之Zuul过滤器实现登录鉴权实战](https://www.cnblogs.com/dalianpai/p/11710142.html)

[视频：狂神说Spring Cloud](https://www.bilibili.com/video/BV1jJ411S7xr)

[狂神说SpringCloud学习笔记](https://blog.csdn.net/weixin_43591980/article/details/106255122#t1)

