# SpringBoot+Mybatis保证读写事务隔离性的三种实现方式

> 实际开发中经常会有这样的需求，注册用户，如果用户名存在则失败，否则注册成功。
>
> 在单线程下，逻辑很简单，但是高并发下需要保证事务隔离性，这里举一个简化版的例子来讲述自己的实现方法。

[TOC]



## 问题

在实际开发的时候，我们经常会做这种事情：

1. 先查询数据库中的数据，得到一些临时结果
2. 根据一些临时结果做判断，进行增删改查操作

也就是说，**第二个阶段的增删改查操作依赖于在第一个阶段的结果**

举个例子，我们的表结构很简单

![image-20210302163200742](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210302163200742.png)

> 查询是否存在dname=TEST的部门，如果不存在插入一个部门名为TEST，如果存在则不操作。
>
> 要求必须不能使数据库中存在两个dname相同的行

我们知道**SpringBoot中的Controller、Service都是单例的**，在实际环境下，面对高并发量的请求，每个请求会起一个线程来进行操作，那么就会发生一些问题

举一个类似“脏读”的例子

```java
@Service
public class DeptServiceImpl implements DeptService{
    @Autowired
    DeptDAO deptDAO;

    @Override
    public int concurrent() {
        Dept dept = deptDAO.queryByName("TEST");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (dept == null) {
            System.out.println("不存在TEST，插入");
            Dept nDept = new Dept().setDName("TEST");
            deptDAO.addDept(nDept);
        } else {
            System.out.println("存在");
        }
        return 1;
    }
}
```

如果不加以处理，我们对以下接口连着发两个请求调用这个方法试试

```java
    @PostMapping("/concurrent")
    public int concurrent(){
        return deptService.concurrent();
    }
```

结果明显是有问题的

![image-20210302163724061](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210302163724061.png)

![image-20210302163758834](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210302163758834.png)

## 方法一：加synchronized锁

最简单方法，牺牲一些性能，加synchronized锁即可

```java
    @Override
    public synchronized int concurrent() {
        Dept dept = deptDAO.queryByName("TEST");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (dept == null) {
            System.out.println("不存在TEST，插入");
            Dept nDept = new Dept().setDName("TEST");
            deptDAO.addDept(nDept);
        } else {
            System.out.println("存在");
        }
        return 1;
    }
```

执行结果没问题

![image-20210302163933812](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210302163933812.png)

注意，这里为了实验使用了`Thread.sleep(3000);`，切记不能使用`object.wait(3000);`，因为wait会释放锁

## 方法二：使用dual表写sql

出现这个问题的原因在于，我们使用了两次mapper，是否能在一个语句里实现需求呢？

其实是可以的，使用dual表

修改Mybatis语句

```xml
    <insert id="addDept" parameterType="com.cpaulyz.PO.Dept">
        insert into dept(dname, db_source) select #{dName},DATABASE() from dual where not exists(
          select * from dept where dname = #{dName}
        );
    </insert>
```

实际上就是

```mysql
insert into dept(dname, db_source) select "TEST",DATABASE() from dual where not exists(
  select * from dept where dname = "TEST"
);
```

然后直接插入即可

```java
    @Override
    public synchronized int concurrent() {
        Dept nDept = new Dept().setDName("TEST");
        deptDAO.addDept(nDept);
        return 1;
    }
```

## 方法三：行锁+@Transactional

分析一下出现问题的原因，主要在于`Dept dept = deptDAO.queryByName("TEST");`时，默认使用的是快照读，即select * from dept where xxxx;

我们可以进行当前读，类似MySQL的锁策略

修改mybatis映射

```xml
    <select id="queryByName" resultType="com.cpaulyz.PO.Dept" resultMap="DeptMap">
        select * from dept where dname=#{name} for update;
    </select>
```

在方法头上加上`@Transactional`注解（该注解还可以进行隔离级别的配置，这里不再赘述）

```java
    @Override
    @Transactional
    public  int concurrent() {
        Dept dept = deptDAO.queryByName("TEST");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (dept == null) {
            System.out.println("不存在TEST，插入");
            Dept nDept = new Dept().setDName("TEST");
            deptDAO.addDept(nDept);
        } else {
            System.out.println("存在");
        }
        return 1;
    }
```

测试，成功

![image-20210302171607298](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210302171607298.png)