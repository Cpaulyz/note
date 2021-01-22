

# JAVA基础：注解应用示例

[TOC]

本文需要的背景知识：[JAVA基础：注解机制](https://www.cnblogs.com/cpaulyz/p/14315973.html)

## 1 利用反射，构建框架

—— 程序员 A : 我写了一个类，它的名字叫做 NoBug，因为它所有的方法都没有错误。 
 —— 我：自信是好事，不过为了防止意外，让我测试一下如何？ 
 —— 程序员 A: 怎么测试？ 
 —— 我：把你写的代码的方法都加上 @Check 这个注解就好了。 
 —— 程序员 A: 好的

* 编写注解

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Check {
    }
    ```

* 实现NoBug类

	```java
	public class NoBug {
	
	    @Check
	    public void suanShu(){
	        System.out.println("1234567890");
	    }
	    @Check
	    public void jiafa(){
	        System.out.println("1+1="+1+1);
	    }
	    @Check
	    public void jiefa(){
	        System.out.println("1-1="+(1-1));
	    }
	    @Check
	    public void chengfa(){
	        System.out.println("3 x 5="+ 3*5);
	    }
	    @Check
	    public void chufa(){
	        System.out.println("6 / 0="+ 6 / 0);
	    }
	
	    public void ziwojieshao(){
	        System.out.println("我写的程序没有 bug!");
	    }
	}
	```

* 编写Test类进行检查

    ```java
    public class TestTool {
    
        public static void main(String[] args) {
            // TODO Auto-generated method stub
            NoBug testobj = new NoBug();
            Class clazz = testobj.getClass();
            Method[] method = clazz.getDeclaredMethods();
            //用来记录测试产生的 log 信息
            StringBuilder log = new StringBuilder();
            // 记录异常的次数
            int errornum = 0;
    
            for ( Method m: method ) {
                // 只有被 @Check 标注过的方法才进行测试
                if ( m.isAnnotationPresent( Check.class )) {
                    try {
                        m.setAccessible(true);
                        m.invoke(testobj, null);
                    } catch (Exception e) {
                        // TODO Auto-generated catch block
                        //e.printStackTrace();
                        errornum++;
                        log.append(m.getName());
                        log.append(" ");
                        log.append("has error:");
                        log.append("\n\r  caused by ");
                        //记录测试过程中，发生的异常的名称
                        log.append(e.getCause().getClass().getSimpleName());
                        log.append("\n\r");
                        //记录测试过程中，发生的异常的具体信息
                        log.append(e.getCause().getMessage());
                        log.append("\n\r");
                    }
                }
            }
            log.append(clazz.getSimpleName());
            log.append(" has  ");
            log.append(errornum);
            log.append(" error.");
            // 生成测试报告
            System.out.println(log.toString());
        }
    }
    ```

输出结果为

![image-20210122221204720](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122221204720.png)

> 参考：https://blog.csdn.net/walk_man_3/article/details/79480326

## 2 AOP日志

### 2.1 MyLog注解

```java
@Target(ElementType.METHOD) //注解放置的目标位置,METHOD是可注解在方法级别上
@Retention(RetentionPolicy.RUNTIME) //注解在哪个阶段执行
@Documented //生成文档
public @interface MyLog {
    String value() default "";
}
```

### 2.2 切面配置类

```java
@Aspect
@Component
public class LogAspect {
    /**
     * 配置织入点 - 自定义注解的包路径
     *
     */
    @Pointcut("@annotation(Annotation.example.MyLog)")
    public void logPointCut() {
    }

    /**
     * 处理完请求后执行
     *
     * @param joinPoint 切点
     */
    @AfterReturning(pointcut = "logPointCut()", returning = "jsonResult")
    public void doAfterReturning(JoinPoint joinPoint, Object jsonResult) {
        System.out.println("切面。。。。。");
        //保存日志

        //从切面织入点处通过反射机制获取织入点处的方法
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //获取切入点所在的方法
        Method method = signature.getMethod();

        //获取操作
        MyLog myLog = method.getAnnotation(MyLog.class);
        if (myLog != null) {
            String value = myLog.value();
            System.out.println("operation "+value);
        }

        //获取请求的类名
        String className = joinPoint.getTarget().getClass().getName();
        //获取请求的方法名
        String methodName = method.getName();
        System.out.println(className + "." + methodName);

        //请求的参数
        Object[] args = joinPoint.getArgs();
        //将参数所在的数组转换成json
        String params = JSON.toJSONString(args);
        System.out.println("params: "+params);
        System.out.println("将日志记录到数据库");
    }

    /**
     * 拦截异常操作
     *
     * @param joinPoint 切点
     * @param e 异常
     */
    @AfterThrowing(value = "logPointCut()", throwing = "e")
    public void doAfterThrowing(JoinPoint joinPoint, Exception e) {
        System.out.println("exception");
    }
}
```

### 2.3 测试类

```java
@RestController
public class LogTest {

    @MyLog("update")
    @PostMapping("/test")
    public void update(){
        System.out.println("update");
    }
}
```

### 2.4 测试

启动服务，使用postman模拟一个请求，可以观察到

![image-20210122232314271](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122232314271.png)

通过这种方法，就可以很好地进行日志记录了

### 2.5 附：maven文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.cpaulyz</groupId>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <artifactId>com.cpaulyz</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>2.23.4</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sourceforge.nekohtml</groupId>
            <artifactId>nekohtml</artifactId>
            <version>1.9.18</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 参考

https://www.pdai.tech/md/java/basic/java-basic-x-annotation.html

