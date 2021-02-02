# SpringCloud(一)：基础知识

[TOC]



## 1 基础知识

### 1.1 微服务的四个问题

1. 服务如何访问？
2. 服务之间如何通信？
3. 服务如何治理？
4. 服务挂了怎么办？

### 1.2 解决方案：SpringCloud生态

* Spring Cloud NetFlix
	* 一站式解决方案
		* API网关：zuul组件
		* 通信：Feign
		* 服务注册发现：Euraka
		* 熔断机制：Hystrix
* Apache Dubbo Zookeeper
	* 半自动，需要整合别人
		* API：没有，找第三方组件
		* 通信：Dubbo
		* 服务注册发现：Zookeeper
		* 熔断机制：没用
* Spring Cloud Alibaba
	* 最新的一站式解决方案

### 1.3 万变不离其宗

1. API
2. HTTP RPC
3. 注册与发现
4. 熔断机制

归其本质：分布式之间的网络不可靠

## 2 微服务

> 建议阅读[什么是微服务架构？](https://www.zhihu.com/question/65502802)

### 2.1 概念

* 微服务架构是一种架构模式，提倡将单一的应用程序划分成为一组小的服务
* 每个服务和运行在自己的进程中，服务之间互相协调、互相配置
* 服务之间采用轻量级的通信机制互相沟通，每个服务围绕具体的业务进行构建，并且能够被独立地部署到生产环境中
* 有一个非常轻量级的集中式管理来协调这些服务
* 可以使用不同的语言来编写服务，也可以使用不同的数据库

目的：解耦！

### 2.2 微服务和微服务架构

* 微服务
	* 强调服务的大小，关注的是某一个点，相当于一个module
* 微服务架构
	* 是一种架构模式

### 2.3 微服务的优缺点

**优点：**

* 单一职责
* 每个服务足够内聚，足够小，代码容易理解，能够聚焦一个业务功能
* 开发简单，效率高
* 松耦合，无法论述在开发还是部署阶段都是独立的
* 可以通过第三方持续集成，如Jenkins
* 纯后端代码
* 每个微服务都有自己的存储能力，可以有自己的数据库，也可以是统一的数据库

**缺点：**

* 开发人员要处理分布式系统的复杂性
* 多服务运维难度随着服务的增加而增大
* 系统部署依赖
* 服务间通信成本
* 数据一致性
* 系统集成测试
* 性能监控
* ...

### 2.4 微服务技术栈

| **微服务技术条目**                     | 落地技术                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| 服务开发                               | SpringBoot、Spring、SpringMVC等                              |
| 服务配置与管理                         | Netfix公司的Archaius、阿里的Diamond等                        |
| 服务注册与发现                         | Eureka、Consul、Zookeeper等                                  |
| 服务调用                               | Rest、PRC、gRPC                                              |
| 服务熔断器                             | Hystrix、Envoy等                                             |
| 负载均衡                               | Ribbon、Nginx等                                              |
| 服务接口调用(客户端调用服务的简化工具) | Fegin等                                                      |
| 消息队列                               | Kafka、RabbitMQ、ActiveMQ等                                  |
| 服务配置中心管理                       | SpringCloudConfig、Chef等                                    |
| 服务路由(API网关)                      | Zuul等                                                       |
| 服务监控                               | Zabbix、Nagios、Metrics、Specatator等                        |
| 全链路追踪                             | Zipkin、Brave、Dapper等                                      |
| 数据流操作开发包                       | SpringCloud Stream(封装与Redis，Rabbit，Kafka等发送接收消息) |
| 时间消息总栈                           | SpringCloud Bus                                              |
| 服务部署                               | Docker、OpenStack、Kubernetes等                              |

## 3 SpringCloud入门概述

### 3.1 SpringCloud是什么？

Spring官网：https://spring.io/

![image-20210202164104251](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202164104251.png)

### 3.2 SpringBoot和SpringCloud的关系

- SpringBoot专注于开苏方便的开发单个个体微服务；
- SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务，整合并管理起来，为各个微服务之间提供：配置管理、服务发现、断路器、路由、为代理、事件总栈、全局锁、决策竞选、分布式会话等等集成服务；
- SpringBoot可以离开SpringCloud独立使用，开发项目，但SpringCloud离不开SpringBoot，属于依赖关系；
- SpringBoot专注于快速、方便的开发单个个体微服务，SpringCloud关注全局的服务治理框架；

### 3.3 Dubbo 和 SpringCloud技术选型

传统架构

![server](https://cyzblog.oss-cn-beijing.aliyuncs.com/server.png)

#### (1) 分布式+服务治理Dubbo

目前成熟的互联网架构，应用服务化拆分 + 消息中间件

#### (2) Dubbo 和 SpringCloud对比

可以看一下社区活跃度：

https://github.com/dubbo

https://github.com/spring-cloud

**对比结果：**

|              | Dubbo         | SpringCloud                  |
| ------------ | ------------- | ---------------------------- |
| 服务注册中心 | Zookeeper     | Spring Cloud Netfilx Eureka  |
| 服务调用方式 | RPC           | REST API                     |
| 服务监控     | Dubbo-monitor | Spring Boot Admin            |
| 断路器       | 不完善        | Spring Cloud Netfilx Hystrix |
| 服务网关     | 无            | Spring Cloud Netfilx Zuul    |
| 分布式配置   | 无            | Spring Cloud Config          |
| 服务跟踪     | 无            | Spring Cloud Sleuth          |
| 消息总栈     | 无            | Spring Cloud Bus             |
| 数据流       | 无            | Spring Cloud Stream          |
| 批量任务     | 无            | Spring Cloud Task            |

**最大区别：Spring Cloud 抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式**

严格来说，这两种方式各有优劣。虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这个优点在当下强调快速演化的微服务环境下，显得更加合适。

**总结：**二者解决的问题域不一样：Dubbo的定位是一款RPC框架，而SpringCloud的目标是微服务架构下的一站式解决方案。

### 3.4 学习网站

- SpringCloud Netflix 中文文档：https://springcloud.cc/spring-cloud-netflix.html
- SpringCloud 中文API文档(官方文档翻译版)：https://springcloud.cc/spring-cloud-dalston.html
- SpringCloud中国社区：http://springcloud.cn/
- SpringCloud中文网：https://springcloud.cc

## 参考

狂神说：https://www.bilibili.com/video/BV1jJ411S7xr

