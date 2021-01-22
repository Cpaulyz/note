# JAVA基础：注解机制

[TOC]



## 1 注解基础

注解是JDK1.5版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。它主要的作用有以下四方面：

- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

这么来说是比较抽象的，我们具体看下注解的常见分类：

- **Java自带的标准注解**，包括`@Override`、`@Deprecated`和`@SuppressWarnings`，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。
- **元注解**，元注解是用于定义注解的注解，包括`@Retention`、`@Target`、`@Inherited`、`@Documented`，`@Retention`用于标明注解被保留的阶段，`@Target`用于标明注解使用的范围，`@Inherited`用于标明注解可继承，`@Documented`用于标明是否生成javadoc文档。
- **自定义注解**，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解。

接下来我们通过这个分类角度来理解注解

## 2 Java内置注解

Java 1.5开始自带的标准注解，包括`@Override`、`@Deprecated`和`@SuppressWarnings`：

- `@Override`：表示当前的方法定义将覆盖父类中的方法
- `@Deprecated`：表示代码被弃用，如果使用了被@Deprecated注解的代码则编译器将发出警告
- `@SuppressWarnings`：表示关闭编译器警告信息

### 2.1 @Override

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

从它的定义我们可以看到，这个注解可以被用来修饰方法，并且它只在编译时有效，在编译后的class文件中便不再存在。这个注解的作用我们大家都不陌生，那就是告诉编译器被修饰的方法是重写的父类的中的相同签名的方法，编译器会对此做出检查，若发现父类中不存在这个方法或是存在的方法签名不同，则会报错

### 2.2 @Deprecated

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE})
public @interface Deprecated {
    /**
     * Returns the version in which the annotated element became deprecated.
     * The version string is in the same format and namespace as the value of
     * the {@code @since} javadoc tag. The default value is the empty
     * string.
     *
     * @return the version string
     * @since 9
     */
    String since() default "";

    /**
     * Indicates whether the annotated element is subject to removal in a
     * future version. The default value is {@code false}.
     *
     * @return whether the element is subject to removal
     * @since 9
     */
    boolean forRemoval() default false;
}
```

它会被文档化，能够保留到运行时，能够修饰构造方法、属性、局部变量、方法、包、参数、类型。这个注解的作用是告诉编译器被修饰的程序元素已被“废弃”，不再建议用户使用

### 2.3 @SuppressWarnings

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}

```

它能够修饰的程序元素包括类型、属性、方法、参数、构造器、局部变量，只能存活在源码时，取值为String[]。它的作用是告诉编译器忽略指定的警告信息，它可以取的值如下所示

著作权归https://www.pdai.tech所有。 链接：https://www.pdai.tech/md/java/basic/java-basic-x-annotation.html



| 参数                     | 作用                                               | 原描述                                                       |
| ------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| all                      | 抑制所有警告                                       | to suppress all warnings                                     |
| boxing                   | 抑制装箱、拆箱操作时候的警告                       | to suppress warnings relative to boxing/unboxing operations  |
| cast                     | 抑制映射相关的警告                                 | to suppress warnings relative to cast operations             |
| dep-ann                  | 抑制启用注释的警告                                 | to suppress warnings relative to deprecated annotation       |
| deprecation              | 抑制过期方法警告                                   | to suppress warnings relative to deprecation                 |
| fallthrough              | 抑制确在switch中缺失breaks的警告                   | to suppress warnings relative to missing breaks in switch statements |
| finally                  | 抑制finally模块没有返回的警告                      | to suppress warnings relative to finally block that don’t return |
| hiding                   | 抑制与隐藏变数的区域变数相关的警告                 | to suppress warnings relative to locals that hide variable（） |
| incomplete-switch        | 忽略没有完整的switch语句                           | to suppress warnings relative to missing entries in a switch statement (enum case) |
| nls                      | 忽略非nls格式的字符                                | to suppress warnings relative to non-nls string literals     |
| null                     | 忽略对null的操作                                   | to suppress warnings relative to null analysis               |
| rawtype                  | 使用generics时忽略没有指定相应的类型               | to suppress warnings relative to un-specific types when using |
| restriction              | 抑制与使用不建议或禁止参照相关的警告               | to suppress warnings relative to usage of discouraged or     |
| serial                   | 忽略在serializable类中没有声明serialVersionUID变量 | to suppress warnings relative to missing serialVersionUID field for a serializable class |
| static-access            | 抑制不正确的静态访问方式警告                       | to suppress warnings relative to incorrect static access     |
| synthetic-access         | 抑制子类没有按最优方法访问内部类的警告             | to suppress warnings relative to unoptimized access from inner classes |
| unchecked                | 抑制没有进行类型检查操作的警告                     | to suppress warnings relative to unchecked operations        |
| unqualified-field-access | 抑制没有权限访问的域的警告                         | to suppress warnings relative to field access unqualified    |
| unused                   | 抑制没被使用过的代码的警告                         | to suppress warnings relative to unused code                 |

## 3 元注解

上述内置注解的定义中使用了一些元注解（注解类型进行注解的注解类），在JDK 1.5中提供了4个标准的元注解：`@Target`，`@Retention`，`@Documented`，`@Inherited`, 在JDK 1.8中提供了两个元注解 `@Repeatable`和`@Native`

### 3.1 @Target

> 作用：用来说明那些被它所注解的注解类可修饰的对象范围

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

其中的ElementType枚举类为

```java
public enum ElementType {
    TYPE, // 类、接口、枚举类
    FIELD, // 成员变量（包括：枚举常量）
    METHOD, // 成员方法
    PARAMETER, // 方法参数
    CONSTRUCTOR, // 构造方法
    LOCAL_VARIABLE, // 局部变量
    ANNOTATION_TYPE, // 注解类
    PACKAGE, // 可用于修饰：包
    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
}
```

### 3.2 @Retention & @RetentionTarget

> 作用：Reteniton注解用来限定那些被它所注解的注解类在注解到其他类上以后，可被保留到何时

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    RetentionPolicy value();
}

```

一共有三种策略，定义在RetentionPolicy枚举中

```java
public enum RetentionPolicy {
    SOURCE,    // 源文件保留
    CLASS,       // 编译期保留，默认值
    RUNTIME   // 运行期保留，可通过反射去获取注解信息
}
```

为了区分，我们做三个注解

```java
@Retention(RetentionPolicy.SOURCE)
public @interface SourcePolicy {
 
}
@Retention(RetentionPolicy.CLASS)
public @interface ClassPolicy {
 
}
@Retention(RetentionPolicy.RUNTIME)
public @interface RuntimePolicy {
 
}

public class RetentionTest {
 
	@SourcePolicy
	public void sourcePolicy() {
	}
 
	@ClassPolicy
	public void classPolicy() {
	}
 
	@RuntimePolicy
	public void runtimePolicy() {
	}
}
```

执行

* `javac RetentionTest.java ClassPolicy.java RuntimePolicy.java SourcePolicy.java`
* `javap -verbose RetentionTest.java`

```java
{
  public Anno.RetentionTest();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public void sourcePolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 7: 0

  public void classPolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 11: 0
    RuntimeInvisibleAnnotations:
      0: #14()
        Anno.ClassPolicy

  public void runtimePolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 15: 0
    RuntimeVisibleAnnotations:
      0: #17()
        Anno.RuntimePolicy
        
}
SourceFile: "RetentionTest.java"

```

从 RetentionTest 的字节码内容我们可以得出以下两点结论：

- 编译器并没有记录下 sourcePolicy() 方法的注解信息；
- 编译器分别使用了 `RuntimeInvisibleAnnotations` 和 `RuntimeVisibleAnnotations` 属性去记录了`classPolicy()`方法 和 `runtimePolicy()`方法 的注解信息

### 3.3 @Documented

> 作用：描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息

```java
@Documented
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface TestDocAnnotation {
 
	public String value() default "default";
}
```

```java
@TestDocAnnotation("myMethodDoc")
public void testDoc() {

}
```

在IDEA中生成JavaDoc

![image-20210122213154950](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122213154950.png)

1. 有`@Documented`注解的情况

	![image-20210122213043875](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122213043875.png)

2.  没有`@Documented`注解的情况

	![image-20210122213113992](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122213113992.png)

### 3.4 @Inherited

> 作用：被它修饰的Annotation将具有继承性。如果某个类使用了被@Inherited修饰的Annotation，则其子类将自动具有该注解

定义注解

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface TestInheritedAnnotation {
    String [] values();
    int number();
}
```

测试

```java
@TestInheritedAnnotation(values = "test",number = 10)
public class Person {
}

class Student extends Person{
    public static void main(String[] args) {
        Class clazz = Student.class;
        Annotation[] annotations = clazz.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation.toString());
        }
    }
}
```

![image-20210122213641971](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210122213641971.png)

即使Student类没有显示地被注解`@TestInheritedAnnotation`，但是它的父类Person被注解，而且`@TestInheritedAnnotation`被`@Inherited`注解，因此Student类自动有了该注解

### 3.5 @Native

使用 @Native 注解修饰成员变量，则表示这个变量可以被本地代码引用，常常被代码生成工具使用。对于 @Native 注解不常使用，了解即可

### 3.6 @Repeatable

JDK8新增

> 作用：允许在同一申明类型(类，属性，或方法)的多次使用同一个注解

java 8之前也有重复使用注解的解决方案，但可读性不是很好，比如下面的代码:

```java
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseOldVersion {

    @Authorities({@Authority(role="Admin"),@Authority(role="Manager")})
    public void doSomeThing(){
    }
}
```

由另一个注解来存储重复注解，在使用时候，用存储注解Authorities来扩展重复注解。

我们再来看看java 8里面的做法:

```java
@Repeatable(Authorities.class)
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseNewVersion {
    @Authority(role="Admin")
    @Authority(role="Manager")
    public void doSomeThing(){ }
}
```

不同的地方是，创建重复注解Authority时，加上@Repeatable,指向存储注解Authorities，在使用时候，直接可以重复使用Authority注解。从上面例子看出，java 8里面做法更适合常规的思维，可读性强一点

## 4 注解与反射接口

> 定义注解后，如何获取注解中的内容呢？反射包java.lang.reflect下的AnnotatedElement接口提供这些方法。这里注意：只有注解被定义为RUNTIME后，该注解才能是运行时可见，当class文件被装载时被保存在class文件中的Annotation才会被虚拟机读取。

AnnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的方法来访问Annotation信息。我们看下具体的先关接口

- `boolean isAnnotationPresent(Class<?extends Annotation> annotationClass)`

判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false。注意：此方法会忽略注解对应的注解容器。

- `<T extends Annotation> T getAnnotation(Class<T> annotationClass)`

返回该程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。

- `Annotation[] getAnnotations()`

返回该程序元素上存在的所有注解，若没有注解，返回长度为0的数组。

- `<T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass)`

返回该程序元素上存在的、指定类型的注解数组。没有注解对应类型的注解时，返回长度为0的数组。该方法的调用者可以随意修改返回的数组，而不会对其他调用者返回的数组产生任何影响。`getAnnotationsByType`方法与 `getAnnotation`的区别在于，`getAnnotationsByType`会检测注解对应的重复注解容器。若程序元素为类，当前类上找不到注解，且该注解为可继承的，则会去父类上检测对应的注解。

- `<T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)`

返回直接存在于此元素上的所有注解。与此接口中的其他方法不同，该方法将忽略继承的注释。如果没有注释直接存在于此元素上，则返回null

- `<T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass)`

返回直接存在于此元素上的所有注解。与此接口中的其他方法不同，该方法将忽略继承的注释

- `Annotation[] getDeclaredAnnotations()`

返回直接存在于此元素上的所有注解及注解对应的重复注解容器。与此接口中的其他方法不同，该方法将忽略继承的注解。如果没有注释直接存在于此元素上，则返回长度为零的一个数组。该方法的调用者可以随意修改返回的数组，而不会对其他调用者返回的数组产生任何影响

## 5 自定义注解

* 自定义注解

	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface MyMethodAnnotation {
	
	    public String title() default "";
	
	    public String description() default "";
	
	}
	```

* 使用注解

	```java
	public class TestMethodAnnotation {
	    @Override
	    @MyMethodAnnotation(title = "toStringMethod", description = "override toString method")
	    public String toString() {
	        return "Override toString method";
	    }
	
	    @Deprecated
	    @MyMethodAnnotation(title = "old static method", description = "deprecated old static method")
	    public static void oldMethod() {
	        System.out.println("old method, don't use it.");
	    }
	
	    @SuppressWarnings({"unchecked", "deprecation"})
	    @MyMethodAnnotation(title = "test method", description = "suppress warning static method")
	    public static void genericsTest() throws FileNotFoundException {
	        List l = new ArrayList();
	        l.add("abc");
	        oldMethod();
	    }
	}
	```

* 添加main方法

	```java
	    public static void main(String[] args) {
	        try {
	            // 获取所有methods
	            Method[] methods = TestMethodAnnotation.class.getClassLoader()
	                    .loadClass(("Annotation.TestMethodAnnotation"))
	                    .getMethods();
	
	            // 遍历
	            for (Method method : methods) {
	                // 方法上是否有MyMethodAnnotation注解
	                if (method.isAnnotationPresent(MyMethodAnnotation.class)) {
	                    try {
	                        // 获取并遍历方法上的所有注解
	                        for (Annotation anno : method.getDeclaredAnnotations()) {
	                            System.out.println("Annotation in Method '"
	                                    + method + "' : " + anno);
	                        }
	
	                        // 获取MyMethodAnnotation对象信息
	                        MyMethodAnnotation methodAnno = method
	                                .getAnnotation(MyMethodAnnotation.class);
	
	                        System.out.println(methodAnno.title());
	
	                    } catch (Throwable ex) {
	                        ex.printStackTrace();
	                    }
	                }
	            }
	        } catch (SecurityException | ClassNotFoundException e) {
	            e.printStackTrace();
	        }
	    }
	```

* 运行结果

	```
	Annotation in Method 'public java.lang.String Annotation.TestMethodAnnotation.toString()' : @Annotation.MyMethodAnnotation(title="toStringMethod", description="override toString method")
	toStringMethod
	Annotation in Method 'public static void Annotation.TestMethodAnnotation.genericsTest() throws java.io.FileNotFoundException' : @Annotation.MyMethodAnnotation(title="test method", description="suppress warning static method")
	test method
	Annotation in Method 'public static void Annotation.TestMethodAnnotation.oldMethod()' : @java.lang.Deprecated(forRemoval=false, since="")
	Annotation in Method 'public static void Annotation.TestMethodAnnotation.oldMethod()' : @Annotation.MyMethodAnnotation(title="old static method", description="deprecated old static method")
	old static method
	```

## 6 注解底层原理

https://blog.csdn.net/qq_20009015/article/details/106038023

https://www.race604.com/annotation-processing/

可能需要一些JVM知识

## 参考

https://www.pdai.tech/md/java/basic/java-basic-x-annotation.html

https://blog.csdn.net/walk_man_3/article/details/79480326