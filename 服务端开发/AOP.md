# AOP

## 切点表达式

原本需要重复定义切点，如下

```java
    @Before("execution(* concert.Performance.perform( .. ))")
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }

    @Before("execution(* concert.Performance.perform( .. ))")
    public void takeSeats() {
        System.out.println("Taking seats");
    }
```

使用@Pointcut

```java
    @Pointcut("execution(* concert.Performance.perform( .. ))")
    public void performance() {
    }

    @Before("performance()")
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }

    @Before("performance()")
    public void takeSeats() {
        System.out.println("Taking seats");
    }

    @AfterReturning("performance()")
    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }

    @AfterThrowing("performance()")
    public void demandRefund() {
        System.out.println("Demand a refund");
    }
```

### 切点指示器之获取参数

通过切点表达式，还可以把参数取下来

```java
    @Pointcut(
            "execution(* soundsystem.CompactDisc.playTrack( int )) " +
                    "&& args(trackNumber)")
    public void trackPlayed(int trackNumber) {
    }
```

* 切在`playTrack( int )`
* `args(trackNumber)`把参数当作trackNumber取下来

之后就可以

```java
    @Before("trackPlayed(trackNumber)")
    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }
```

### 切点指示器之限定bean

```java
    @Pointcut(
            "execution(* soundsystem.CompactDisc.playTrack( int )) " +
                    "&&bean(sgtPeppers) && args(trackNumber)")
    public void trackPlayed(int trackNumber) {
    }
```

## 引入

```java
@Aspect
public class EncoreableIntroducer {
    @DeclareParents(value = "concert.Performance+",//后面的+表示应用到所有实现了该接口的Bean
            defaultImpl = DefaultEncoreable.class)
    public static Encoreable encoreable;
}
```

为所有实现了该接口的Bean的添加新的行为

增加的新行为是`DefaultEncoreable.class`的类的行为

> 问题：
>
> 我们没有手动实例化DefaultEncoreable，是Spring帮我们进行实例化。
>
> 如果有多个类实现了Performance接口，那么Spring会为我们实例化几个DefaultEncoreable类？
>
> 答案应该是0？？
>
> ```java
> public class DefaultEncoreable implements Encoreable {
> 
>     public static int count = 0;
> 
>     DefaultEncoreable(){
>         count++;
>         System.out.println("DefaultEncoreable实例化："+count);
>     }
> 
>     public void performEncore() {
>         System.out.println("perform the encore!");
>     }
> 
> }
> ```
>
> 添加count以后，压根没看到输出，静态变量count也一直是0