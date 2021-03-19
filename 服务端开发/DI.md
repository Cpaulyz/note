# DI

## 自动装配

> 注解形式，侵入式的

### 1

`@Component` + `@Autowired`

需要告诉Spring扫描哪些包

```java
@Configuration
@ComponentScan
public class CDPlayerConfig {
}
```

`@ComponentScan`

* basePackages：字符串，扫描包和子包
* basePackagesClasses：.class，扫描其所在包和子包

`System.getProperty("line.separator")`

用于适应不同的操作系统

### 2

`@Bean`

```java
@Configuration
public class CDPlayerConfig {

    @Bean
    public CompactDisc compactDisc() {
        return new SgtPeppers();
    }

    @Bean
    public CDPlayer cdPlayer(CompactDisc cd) {
        return new CDPlayer(cd);
    }

}
```

以加Bean的方式注入，加在方法上，返回一个对象

## XML装配

> 非侵入式的

```xml
  <bean id="cdPlayer" class="soundsystem.CDPlayer"
        c:cd-ref="compactDisc" />
```

在生产环境中很危险，因为c:cd-ref是高耦合的

建议改为

```xml
  <bean id="cdPlayer" class="soundsystem.CDPlayer"
        c:_0-ref="compactDisc" />
```

_0代表是第0个参数

[spring配置文件中的p和c命名空间的使用](https://blog.csdn.net/wqh0830/article/details/86188208)

