# JAVA并发(四)：关键词volatile

[TOC]

> 整理自
>
> https://www.pdai.tech/md/java/thread/java-thread-x-key-volatile.html
>
> https://blog.csdn.net/luman1991/article/details/52861567

## 1 本文介绍以下问题

- volatile关键字的作用是什么?
- volatile能保证原子性吗?
- 之前32位机器上共享的long和double变量的为什么要用volatile? 现在64位机器上是否也要设置呢?
- i++为什么不能保证原子性?
- volatile是如何实现可见性的?  内存屏障。
- volatile是如何实现有序性的?  happens-before等
- 说下volatile的应用场景?

## 2 volatile的作用

### 2.1 防重排序

我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁(DCL)的方式来实现。其源码如下

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

现在我们分析一下为什么要在变量singleton之间加上volatile关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤：

1. 分配内存空间。

2. 初始化对象。

3. 将内存空间的地址赋值给对应的引用。

但是由于操作系统可以`对指令进行重排序`，所以上面的过程也可能会变成如下过程：

1. 分配内存空间。

2. 将内存空间的地址赋值给对应的引用。

3. 初始化对象

如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。

### 2.2 实现可见性

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。volatile关键字能有效的解决这个问题

```java
public class VolatileTest {
    int a = 1;
    int b = 2;

    public void change(){
        a = 3;
        b = a;
    }

    public void print(){
        System.out.println("b="+b+";a="+a);
    }

    public static void main(String[] args) {
        while (true){
            final VolatileTest test = new VolatileTest();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }
}
```

直观上说，这段代码的结果只可能有两种：b=3;a=3 或 b=2;a=1。不过运行上面的代码(可能时间上要长一点)，你会发现除了上两种结果之外，还出现了第三种结果：

![image-20210120171943974](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210120171943974.png)

为什么会出现b=3;a=1这种结果呢? 正常情况下，如果先执行change方法，再执行print方法，输出结果应该为b=3;a=3。相反，如果先执行的print方法，再执行change方法，结果应该是 b=2;a=1。那b=3;a=1的结果是怎么出来的? 原因就是第一个线程将值a=3修改后，但是对第二个线程是不可见的，所以才出现这一结果。如果将a和b都改成volatile类型的变量再执行，则再也不会出现b=3;a=1的结果了

### 3.2 保证原子性：单次读写

1. **i++为什么不能保证原子性?**

	对于原子性，需要强调一点，也是大家容易误解的一点：对volatile变量的单次读/写操作可以保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作

	* 读取i的值。

	* 对i加1。

	* 将i的值写回内存。 volatile是无法保证这三个操作是具有原子性的，我们可以通过AtomicInteger或者Synchronized来保证+1操作的原子性。 注：上面几段代码中多处执行了Thread.sleep()方法，目的是为了增加并发问题的产生几率，无其他作用

2. **共享的long和double变量要用volatile** 

	因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此普通的long或double类型读/写可能不是原子的。因此，鼓励大家将共享的long和double变量设置为volatile类型，这样能保证任何情况下对long和double的单次读/写操作都具有原子性

## 4 volatile实现原理

### 4.1 可见性实现

> volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现:

* 内存屏障，又称内存栅栏，是一个 CPU 指令。

* 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序

写一段简单的 Java 代码，声明一个 volatile 变量，并赋值。

```java
public class Test {
    private volatile int a;
    public void update() {
        a = 1;
    }
    public static void main(String[] args) {
        Test test = new Test();
        test.update();
    }
}
```

通过 hsdis 和 jitwatch 工具可以得到编译后的汇编代码:

```bash
......
  0x0000000002951563: and    $0xffffffffffffff87,%rdi
  0x0000000002951567: je     0x00000000029515f8
  0x000000000295156d: test   $0x7,%rdi
  0x0000000002951574: jne    0x00000000029515bd
  0x0000000002951576: test   $0x300,%rdi
  0x000000000295157d: jne    0x000000000295159c
  0x000000000295157f: and    $0x37f,%rax
  0x0000000002951586: mov    %rax,%rdi
  0x0000000002951589: or     %r15,%rdi
  0x000000000295158c: lock cmpxchg %rdi,(%rdx)  //在 volatile 修饰的共享变量进行写操作的时候会多出 lock 前缀的指令
  0x0000000002951591: jne    0x0000000002951a15
  0x0000000002951597: jmpq   0x00000000029515f8
  0x000000000295159c: mov    0x8(%rdx),%edi
  0x000000000295159f: shl    $0x3,%rdi
  0x00000000029515a3: mov    0xa8(%rdi),%rdi
  0x00000000029515aa: or     %r15,%rdi
......
```

lock 前缀的指令在多核处理器下会引发两件事情:

- 将当前处理器缓存行的数据写回到系统内存。
- 写回内存的操作会使在其他 CPU 里缓存了该内存地址的额数据无效。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存(L1，L2 或其他)后再进行操作，但操作完不知道何时会写到内存。

如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 lock 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。

为了保证各个处理器的缓存是一致的，实现了缓存一致性协议(MESI)，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

所有多核处理器下还会完成：当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

volatile 变量通过这样的机制就使得每个线程都能获得该变量的最新值

### 4.2 有序性实现

1. **happen-before关系**

	happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读

	```java
	//假设线程A执行writer方法，线程B执行reader方法
	class VolatileExample {
	    int a = 0;
	    volatile boolean flag = false;
	    
	    public void writer() {
	        a = 1;              // 1 线程A修改共享变量
	        flag = true;        // 2 线程A写volatile变量
	    } 
	    
	    public void reader() {
	        if (flag) {         // 3 线程B读同一个volatile变量
	        int i = a;          // 4 线程B读共享变量
	        ……
	        }
	    }
	}
	```

	根据 happens-before 规则，上面过程会建立 3 类 happens-before 关系。

	- 根据程序次序规则：1 happens-before 2 且 3 happens-before 4。
	- 根据 volatile 规则：2 happens-before 3。
	- 根据 happens-before 的传递性规则：1 happens-before 4。

	![image-20210120173654218](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210120173654218.png)

2. **volatile禁止重排序**

	为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。

	Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。

	JMM 会针对编译器制定 volatile 重排序规则表。

	![image-20210120173858761](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210120173858761.png)

	为了实现 volatile 内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

	对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM 采取了保守的策略。

	- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。
	- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。
	- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。
	- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。

	volatile 写是在前面和后面分别插入内存屏障，而 volatile 读操作是在后面插入两个内存屏障。

	| 内存屏障        | 说明                                                        |
	| --------------- | ----------------------------------------------------------- |
	| StoreStore 屏障 | 禁止上面的普通写和下面的 volatile 写重排序。                |
	| StoreLoad 屏障  | 防止上面的 volatile 写与下面可能有的 volatile 读/写重排序。 |
	| LoadLoad 屏障   | 禁止下面所有的普通读操作和上面的 volatile 读重排序。        |
	| LoadStore 屏障  | 禁止下面所有的普通写操作和上面的 volatile 读重排序。        |

	![image-20210120174316534](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210120174316534.png)

	![image-20210120174322567](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210120174322567.png)

##  5 volaile实际应用

Volatile 变量具有 `synchronized` 的可见性特性，但是**不具备原子性**。这就是说线程能够自动发现 volatile 变量的最新值。

Volatile 变量可用于提供线程安全，但是只能应用于非常有限的一组用例：**多个变量之间或者某个变量的当前值与修改后值之间没有约束**。因此，单独使用 volatile 还不足以实现计数器、互斥锁或任何具有与多个变量相关的不变式（Invariants）的类（例如 “start <=end”）。

出于简易性或可伸缩性的考虑，您可能倾向于使用 volatile 变量而不是锁。当使用 volatile 变量而非锁时，某些习惯用法（idiom）更加易于编码和阅读。此外，volatile  变量不会像锁那样造成线程阻塞，因此也很少造成可伸缩性问题。在某些情况下，如果读操作远远大于写操作，volatile 变量还可以提供优于锁的**性能**优势

您只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：

- **对变量的写操作不依赖于当前值**
- **该变量没有包含在具有其他变量的不变式中**

第一个条件很好理解，虽然增量操作（`x++`）看上去类似一个单独操作，实际上它是一个由（读取－修改－写入）操作序列组成的组合操作，必须以原子方式执行。因此，第一个条件的限制使 volatile 变量**不能用作线程安全计数器**。

第二个条件，我们可以举一个反例

```java
volatile a = 1;
void method(){
    if(a<2){
        a++;
    }
}
```

有可能两个程序都通过`if(a<2)`代码块，违反了原子性

### 5.1 应用1：状态标志

也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。

```java
volatile boolean shutdownRequested;
......
public void shutdown() { shutdownRequested = true; }
public void doWork() { 
    while (!shutdownRequested) { 
        // do stuff
    }
}
```

这种类型的状态标记的一个公共特性是：**通常只有一种状态转换**；`shutdownRequested` 标志从`false` 转换为`true`，然后程序停止。这种模式可以扩展到来回转换的状态标志，但是只有在转换周期不被察觉的情况下才能扩展（从`false` 到`true`，再转换到`false`）。此外，还需要某些原子状态转换机制，例如原子变量

### 5.2 应用2：一次性安全发布（one-time safe publication）

在缺乏同步的情况下，可能会遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的旧值同时存在。

这就是造成著名的双重检查锁定（double-checked-locking）问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象

```java
 private volatile static Singleton instace;     
    
public static Singleton getInstance(){     
    //第一次null检查       
    if(instance == null){              
        synchronized(Singleton.class) {    //1       
            //第二次null检查         
            if(instance == null){          //2    
                instance = new Singleton();//3    
            }    
        }             
    }    
    return instance; 
```

如果不用volatile，则因为内存模型允许所谓的“无序写入”，可能导致失败。**——某个线程可能会获得一个未完全初始化的实例。**

考察上述代码中的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 instance 来引用此对象。这行代码的问题是：**在Singleton 构造函数体执行之前，变量instance 可能成为非 null 的！**

在解释这个现象如何发生前，请先暂时接受这一事实，我们先来考察一下双重检查锁定是如何被破坏的。假设上述代码执行以下事件序列：

1. 线程 1 进入 getInstance() 方法。
2. 由于 instance 为 null，线程 1 在 //1 处进入synchronized 块。
3. 线程 1 前进到 //3 处，但在构造函数执行之前，使实例成为非null。
4. 线程 1 被线程 2 预占。
5. 线程 2 检查实例是否为 null。因为实例不为 null，线程 2 将instance 引用返回，返回一个构造完整但**部分初始化**了的Singleton 对象。
6. 线程 2 被线程 1 预占。
7. 线程 1 通过运行 Singleton 对象的构造函数并将引用返回给它，来完成对该对象的初始化。

### 5.3 应用3：独立观察（independent observation）

安全使用 volatile 的另一种简单模式是：定期 “发布” 观察结果供程序内部使用。【例如】假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。

使用该模式的另一种应用程序就是收集程序的统计信息。【例】如下代码展示了身份验证机制如何记忆最近一次登录的用户的名字。将反复使用`lastUser` 引用来发布值，以供程序的其他部分使用。

```java
 public class UserManager {  
    public volatile String lastUser; //发布的信息  
  
    public boolean authenticate(String user, String password) {  
        boolean valid = passwordIsValid(user, password);  
        if (valid) {  
            User u = new User();  
            activeUsers.add(u);  
            lastUser = user;  
        }  
        return valid;  
    }  
}  
```

### 5.4 应用4： volatile bean 模式

在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。(这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义)。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。

```java
@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
```

### 5.5 应用5：开销较低的读－写锁策略

如果读操作远远超过写操作，您可以结合使用**内部锁**和 **volatile 变量**来减少公共代码路径的开销。

如下显示的线程安全的计数器，使用 `synchronized` 确保增量操作是原子的，并使用 `volatile` 保证当前结果的可见性。如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。

```java
@ThreadSafe
public class CheesyCounter {
    // Employs the cheap read-write lock trick
    // All mutative operations MUST be done with the 'this' lock held
    @GuardedBy("this") private volatile int value;
 
    public int getValue() { return value; }
 
    public synchronized int increment() {
        return value++;
    }
}
```

