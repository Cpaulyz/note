# SpringCloud(二)：Rest环境搭建

[TOC]

> 本文主要介绍了SpringCloud开发使用的Rest环境搭建，类似SpringBoot，但是作为总项目下的三个Module进行搭建

## 1 项目总结构

![image-20210202213204950](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202213204950.png)

## 2 父项目

创建maven项目，一路next

![image-20210202184510107](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202184510107.png)

把src删掉

![image-20210202184550803](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202184550803.png)

修改pom.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cpaulyz</groupId>
    <artifactId>springcloud-learning</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--打包方式 pom-->
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.18.16</lombok.version>
    </properties>

    <dependencyManagement>
        <dependencies> <!--springCloud的依赖-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--SpringBoot-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--数据库-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.17</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.1.2</version>
            </dependency>
            <!--SpringBoot 启动器-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.2</version>
            </dependency>
            <!--日志测试~-->
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

完成以后，项目结构如下：

![image-20210202213406233](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202213406233.png)

## 3 springboot-api

新建module，选择maven项目

![image-20210202190855021](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202190855021.png)

其中的pom为

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

    <artifactId>springcloud-api</artifactId>

    <!--当前的module自己需要的依赖，如果父依赖中已经有了就不用写了-->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```



创建一个数据库和表

![image-20210202191800058](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202191800058.png)

插入数据

```sql
insert into dept(dname, db_source) values ("开发部",DATABASE());
insert into dept(dname, db_source) values ("人事部",DATABASE());
insert into dept(dname, db_source) values ("财务部",DATABASE());
insert into dept(dname, db_source) values ("市场部",DATABASE());
insert into dept(dname, db_source) values ("运维部",DATABASE());
```

创建Dept实体类

```java
// Dept实体类
@Data
@NoArgsConstructor
@Accessors(chain = true)
public class Dept implements Serializable {
    private int dNo;
    private String dName;
    // 这个服务在哪个数据库的字段
    private String dbSource;

    public Dept(String dName){
        this.dName = dName;
    }
}
```

> 注：
>
> * 要实现Serializable
>
> * 这里用了lombok
>
> * 其中`@Accessors(chain = true)`可以支持链式写法，比如`dept.SetDName(xxx).setDNo(xxx);`

完成以后，module结构如下：

![image-20210202213323395](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202213323395.png)

## 4 springcloud-provider-dept-8001

和前面一样，创建一个Module，选择maven工程，名字为springcloud-provider-dept-8001

就是一个传统的controller-service-data三层的后端

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

    <artifactId>springcloud-provider-dept-8001</artifactId>

    <dependencies>
        <!--需要拿到实体类，需要配置api-module-->
        <dependency>
            <groupId>com.cpaulyz</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
            <version>2.4.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--jetty-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!--热部署工具-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
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
```

DeptDAO.java，对应的mapper文件略

```java
@Mapper
@Repository
public interface DeptDAO {
    public boolean addDept(Dept dept);

    public Dept queryById(int id);

    public List<Dept> queryAll();
}
```

DeptService.java

```java
public interface DeptService {

    public boolean addDept(Dept dept);

    public Dept queryById(int id);

    public List<Dept> queryAll();
}
```

DeptServiceImpl.java

```java
@Service
public class DeptServiceImpl implements DeptService{
    @Autowired
    DeptDAO deptDAO;

    @Override
    public boolean addDept(Dept dept) {
        return deptDAO.addDept(dept);
    }

    @Override
    public Dept queryById(int id) {
        return deptDAO.queryById(id);
    }

    @Override
    public List<Dept> queryAll() {
        return deptDAO.queryAll();
    }
}
```

DeptController.java

```java
@RestController
@RequestMapping("/dept")
public class DeptController {
    @Autowired
    DeptService deptService;

    @PostMapping("/add")
    public boolean addDept(Dept dept){
        return deptService.addDept(dept);
    }

    @GetMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        return deptService.queryById(id);
    }

    @PostMapping("/getAll")
    public List<Dept> getAll(){
        return deptService.queryAll();
    }
}
```

## 5 springcloud-consumer-dept-8002

在父项目下创建module，选择maven，名字为springcloud-consumer-dept-8002

pom.xml

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

    <artifactId>springcloud-consumer-dept-8002</artifactId>

    <dependencies>
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

application.yml

```YML
server:
  port: 8002
```

DeptConsumerController.java

理解: 

* 消费者，不应该有service层
* RestTemplate 供我们直接调用
* 注册到Spring中

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

需要添加RestTemplate为Bean

```java
@Configuration
public class BeanConfig {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

写启动类，启动即可

```java
@SpringBootApplication
public class DeptConsumer_8002 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_8002.class,args);
    }
}
```

完成以后，module结构如下：

![image-20210202213345143](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202213345143.png)

## 6 项目启动

分别启动

* springcloud-provider-dept-8001

* springcloud-consumer-dept-8002

简单测一下

![image-20210202213003941](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210202213003941.png)