# MIT 6.830 LAB1 SimpleDB

[TOC]

> 2021/03/30-2021/03/31

## 前言

课程地址：http://db.lcs.mit.edu/6.830/sched.php

代码：https://github.com/MIT-DB-Class/simple-db-hw

讲义：https://github.com/MIT-DB-Class/course-info-2018/

lab1的实现比较简单，具体代码可以参考reference中的博客或是本人的Github，这里主要梳理一下项目结构

![](https://img-blog.csdnimg.cn/20200103180012189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hqdzE5OTY2Ng==,size_16,color_FFFFFF,t_70)

## LAB1

### exercise1 Fields and Tuples 

> Tuple是数据库中的数据，TupleDesc就是数据库中的schema

![6830lab1ex1](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex1.png)

实现

- src/simpledb/TupleDesc.java
- src/simpledb/Tuple.java

TupleDesc就是数据库中的schema，里面有一组TDItems定义了每一列的类型

Tuple是数据库中的数据，里面的Field就是数据

比较简单，分别在两个类中添加List和一些成员变量即可

### exercise2 Catalog

> Catalog是整个数据库的目录，可以通过tableid获取DbFile

![6830lab1ex2](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex2.png)

实现

* src/simpledb/Catalog.java

Catelog是Table的集合，然后每个Table有`DbFile, PrimaryKeyField, Name`

全局catalog是分配给整个SimpleDB进程的Catalog类一个实例，可以通过方法Database.getCatalog()获得，global buffer pool可以通过方法Database.getBufferPool()获得。

添加以下数据结构后就很好实现了

```java
public class Catalog {

    class Table{
        private DbFile file;
        private String name;
        private String pkeyField;


        public Table(DbFile file, String name, String pkeyField){
            this.file = file;
            this.name = name;
            this.pkeyField = pkeyField;
        }
		// getter and setter
    }
    /* <name,Table>*/
    private ConcurrentHashMap<String,Table> stringTableMap;

    /* <tableId,Table>*/
    private ConcurrentHashMap<Integer,Table> integerTableMap;

    ...
}
```

### exercise3 BufferPool

> BufferPool通过Catalog获取到对应的DbFile，从DbFile中readPage

![6830lab1ex3](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex3.png)

实现

* src/simpledb/BufferPool.java

buffer pool（在SimpleDB中是BufferPool类）负责将内存最近读过的物理页缓存下来。所有的读写操作通过buffer pool读写硬盘上不同文件。

Database类提供了一个静态方法Database.getBufferPool()，返回整个SimpleDB进程的BufferPool实例引用。

这里用到了Page和PageId类，注意到PageId中的hashCode

```java
    /**
     * @return a hash code for this page, represented by the concatenation of
     *   the table number and the page number (needed if a PageId is used as a
     *   key in a hash table in the BufferPool, for example.)
     * @see BufferPool
     */
    public int hashCode();
```

暗示了我们使用hashCode作为key来构建一个hashmap

在Lab1中需要做的只需要添加以下数据结构并实现getPage即可

```java
    private int numPages;
    /**
     * pages in buffer pool
     */
    private ConcurrentHashMap<Integer,Page> pages;
```

### exercise4 HeapPage

> HeapPage是内存中的一个页，包含某个table中的若干个Tuple和TupleDesc；
>
> HeapPageId通过tableid和pageNo来标识一个HeapPage；
>
> RecordId是Page中某个Tuple的标识

![6830lab1ex4](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex4.png)

实现

- src/simpledb/HeapPageId.java
- src/simpledb/RecordID.java
- src/simpledb/HeapPage.java

前两个比较简单，主要注意hashCode的算法，借助String类来实现，以RecordId为例

```java
    /**
     * You should implement the hashCode() so that two equal RecordId instances
     * (with respect to equals()) have the same hashCode().
     * 
     * @return An int that is the same for equal RecordId objects.
     */
    @Override
    public int hashCode() {
        String s = pageId.hashCode()+"#"+tupleNumber;
        return s.hashCode();
    }
```

### exercise5 HeapFile

> HeapFile是DbFile的一个实现，是BufferPool正是通过DbFile来读取Page的；
>
> HeapFile中需要实现一个DbIterator

![6830lab1ex5](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex5.png)

实现

* src/simpledb/HeapFile.java

难点

1. 需要计算File偏移量
2. 不能直接调用BufferPool的readPage方法，要使用RandomAccessFile打开文件进行真正的读，将磁盘文件读进内存中
3. 需要自己实现一个HeapFileIterator类，实现DbFileIterator接口

### exercise6 Operators

> 实现了OpIterator接口，SimpleDP和程序交互的过程中，现在root operator上调用getNext，之后在子节点上继续调用getNext，一直下去，直到leaf operators 被调用。他们从硬盘上读取tuples，并在树结构上传递。

![6830lab1ex6](https://cyzblog.oss-cn-beijing.aliyuncs.com/6830lab1ex6.png)

实现

* src/simpledb/SeqScan.java

调用exercise5中实现的HeapFileIterator即可

## reference

6.830 Lab 1: SimpleDB：https://blog.csdn.net/hjw199666/article/details/103486328

MIT6.830: Database System lab1 note：https://zhuanlan.zhihu.com/p/58595037

6.830 Lab 总览：https://blog.csdn.net/hjw199666/article/details/103824797