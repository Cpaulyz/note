# Swagger2：自动生成API接口文档

[TOC]

> 有个项目需要用到Swagger2来做前后端对接，查了下文档发现挺好用的，记录一下
>
> [Swagger2使用文档](https://gumutianqi1.gitbooks.io/specification-doc/content/tools-doc/spring-boot-swagger2-guide.html)

## 依赖

```xml
<!--swagger2的依赖-->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.9.2</version>
</dependency>
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.9.2</version>
</dependency>
```

## 配置类

```java
@Configuration
@EnableSwagger2
public class Swagger2Conf {
    @Bean
    public Docket getUserDocket(){
        ApiInfo apiInfo=new ApiInfoBuilder()
                .title("Swagger2演示")//api标题
                .description("API文档")//api描述
                .version("1.0.0")//版本号
                .contact("cyz")//本API负责人的联系信息
                .build();
        return new Docket(DocumentationType.SWAGGER_2)//文档类型（swagger2）
                .apiInfo(apiInfo)//设置包含在json ResourceListing响应中的api元信息
                .select()//启动用于api选择的构建器
                .apis(RequestHandlerSelectors.basePackage("com.cpaulyz.controller"))//扫描接口的包
                .paths(PathSelectors.any())//路径过滤器（扫描所有路径）
                .build();
    }
}
```

## 常用注解

| 注解               | 使用的地方     | 用途                                 |
| ------------------ | -------------- | ------------------------------------ |
| @Api               | 类/接口        | 描述类/接口主要用途                  |
| @ApiOperation      | 方法           | 描述方法的用途                       |
| @ApiImplicitParam  | 方法           | 用于描述接口的非对象参数             |
| @ApiImplicitParams | 方法           | 用于描述接口的非对象参数集           |
| @ApiIgnore         | 类/方法/参数   | Swagger 文档不会显示拥有该注解的接口 |
| @ApiModel          | 参数实体类     | 可设置接口相关实体的描述             |
| @ApiModelProperty  | 参数实体类属性 | 可设置实体属性的相关描述             |
| @ApiResponses      | 方法返回值     | 用于描述接口的方法返回值             |
| @ApiResponse       | 方法返回值     | 用于描述接口的方法返回值集           |

### @ApiImplicitParams、@ApiImplicitParam：方法参数的说明

```text
@ApiImplicitParams：用在请求的方法上，包含一组参数说明
    @ApiImplicitParam：对单个参数的说明      
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · body（请求体）-->  @RequestBody User user
            · form（普通表单提交）     
        dataType：参数类型，默认String，其它值dataType="int"       
        defaultValue：参数的默认值
```

单个参数举例

```java
@ApiOperation("根据部门Id删除")
@ApiImplicitParam(name="depId",value="部门id",required=true,paramType="query")
@GetMapping("/delete")
public AjaxResult delete(String depId) {
    //TODO
}
```

### @ApiResponses、@ApiResponse：方法返回值的说明

```text
@ApiResponses：方法返回对象的说明
    @ApiResponse：每个参数的说明
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类
```

## 示例

### 配置实体类

```java
@ApiModel("部门实体类")
@Data
@NoArgsConstructor
@Accessors(chain = true)
public class Dept implements Serializable {
    @ApiModelProperty("ID")
    private int dNo;
    @ApiModelProperty("部门名称")
    private String dName;
    // 这个服务在哪个数据库的字段
    @ApiModelProperty("所属数据库")
    private String dbSource;
}
```

### 配置接口类

```java
@Api(tags = "部门管理api")
@RestController
@RequestMapping("/dept")
public class DeptController {

    @Autowired
    DeptService deptService;

    @ApiOperation("添加部门")
    @PostMapping("/add")
    public boolean addDept(Dept dept){
        return deptService.addDept(dept);
    }

    @ApiOperation(value = "根据ID获取部门",notes = "我是notes")
    @ApiImplicitParams(
            @ApiImplicitParam(name = "id",value = "部门id",required = true)
    )
    @GetMapping("/get/{id}")
    public Dept get(@PathVariable("id") int id){
        return deptService.queryById(id);
    }

    @ApiOperation("获取全部部门")
    @PostMapping("/getAll")
    public List<Dept> getAll(){
        return deptService.queryAll();
    }


    @Autowired
    DiscoveryClient client;

    @ApiIgnore()
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

### 访问UI

`http://localhost:xxxx/swagger-ui.html`

![image-20210204145910901](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210204145910901.png)

> 在webUI上也可以发送请求进行测试，挺好用的

### 导入postman

import->import from link

![image-20210204150129827](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210204150129827.png)

之后就可以方便调试了

![image-20210204150154522](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210204150154522.png)