# MIT 6.830 LAB2 DBOperator

[TOC]

> 2021/04/01-2021/04/05

## 前言

课程地址：http://db.lcs.mit.edu/6.830/sched.php

代码：https://github.com/MIT-DB-Class/simple-db-hw

讲义：https://github.com/MIT-DB-Class/course-info-2018/

清明期间用零散的时间把Lab2写完了，整个Lab2基本是在为simpledb实现一个基本的操作。比如我们常用的选择、join、聚合、插入、删除等。

## LAB2

### exercise1 Join&Filter

> xxxPredicate是辅助类，一般用于根据Field是否满足条件来筛选Tuple；
>
> Filter实现了Operator接口，理解来说就是根据某个Predicate来筛选Tuple，理解来说就是类似筛选出满足where xxx="xxx"的Tuple；
>
> Join实现了Operator接口，是关系运算中的连接操作
>
> Operator接口是实现类可以参考Project.java或者OrderBy.java

 实现

* src/simpledb/Predicate.java 
* src/simpledb/JoinPredicate.java 
* src/simpledb/Filter.java 
* src/simpledb/Join.java

Join难点在于fetchNext，由于是嵌套循环，需要在Join类中添加成员变量`private Tuple t1;`

```java
    protected Tuple fetchNext() throws TransactionAbortedException, DbException {
        while (t1!=null||child1.hasNext()){
            if(t1==null){ // init t1 if null
                if(child1.hasNext()){
                    t1 = child1.next();
                }else{
                    return null;
                }
            }
            if(!child2.hasNext()){ // t2到头了，nextLoop
                if(child1.hasNext()){
                    child2.rewind();
                    t1 = child1.next();
                }else{
                    return null;
                }
            }
            while (child2.hasNext()){
                Tuple t2 = child2.next();
                if(joinPredicate.filter(t1,t2)){
                    Tuple res = new Tuple(getTupleDesc());
                    for (int i = 0; i < t1.getTupleDesc().numFields(); i++) {
                        res.setField(i,t1.getField(i));
                    }
                    for (int i = 0; i < t2.getTupleDesc().numFields(); i++) {
                        res.setField(t1.getTupleDesc().numFields()+i,t2.getField(i));
                    }
                    return res;
                }
            }
        }
        return null;
    }
```

这边的基类Operator其实也是一个OpIterator，其`next`、`hasNext`的实现采用了**模板方法**的设计模式，用了一个Tuple作为缓存，十分巧妙

子类（也就是我们这里实现的Join、Filter）只需要实现fetchNext就可以了

```java
public abstract class Operator implements OpIterator {

    private Tuple next = null;
    
    public boolean hasNext() throws DbException, TransactionAbortedException {
        if (!this.open)
            throw new IllegalStateException("Operator not yet open");
        
        if (next == null)
            next = fetchNext();
        return next != null;
    }

    public Tuple next() throws DbException, TransactionAbortedException,
            NoSuchElementException {
        if (next == null) {
            next = fetchNext();
            if (next == null)
                throw new NoSuchElementException();
        }

        Tuple result = next;
        next = null;
        return result;
    }

    /**
     * Returns the next Tuple in the iterator, or null if the iteration is
     * finished. Operator uses this method to implement both <code>next</code>
     * and <code>hasNext</code>.
     * 
     * @return the next Tuple in the iterator, or null if the iteration is
     *         finished.
     */
    protected abstract Tuple fetchNext() throws DbException,
            TransactionAbortedException;
    
    ....
}
```

### exercise2 Aggregate

> Aggregate实现Operator接口，是聚合操作类。比如select count(*) from xx group by xxx
>
> IntegerAggregator和StringAggregator其实都是Aggregate的辅助类
>
> 区别在于聚合操作字段的属性，如果是Integer，可以有count、sum、max、min、avg操作；如果是String，只有count操作

实现

* src/simpledb/IntegerAggregator.java 
* src/simpledb/StringAggregator.java 
* src/simpledb/Aggregate.java

注意这里的groupby字段可以是空，也就是可以允许有没有groupby的情况下使用聚合函数，这是符合MySQL语法的，如select count(*) from user;

总的来说，难点只在于实现IntegerAggregator，StringAggregator完全可以看作IntegerAggregator的子类，而Aggregate只是对xxxAggregator的一个封装，只需要调用xxxAggregator即可。

这里我的实现和网上大多数代码不一样，充分使用了依赖倒置原则。

以IntegerAggregator为例，我们需要实现五个聚合操作，将其封装为GBHandler私有抽象类，通过**策略模式**进行实现，保证可读性和可修改性。将聚合操作的职责封装进GBHandler中，顶层的IntegerAggregator只需要将其计算后的结果，使用TupleIterator进行封装，返回结果即可。

```java
/**
 * Knows how to compute some aggregate over a set of IntFields.
 *
 * finish in lab2 exercise2
 */
public class IntegerAggregator implements Aggregator {

    private static final long serialVersionUID = 1L;

    private static final String NO_GROUPING_KEY = "NO_GROUPING_KEY";
    /**
     * the 0-based index of the group-by field in the tuple,
     * or NO_GROUPING if there is no grouping
     */
    private int gbFieldIndex;
    /**
     * the type of the group by field (e.g., Type.INT_TYPE),
     * or null if there is no grouping
     */
    private Type gbFieldType;
    /**
     * the 0-based index of the aggregate field in the tuple
     */
    private int aggregateFieldIndex;
    /**
     * the aggregation operator
     */
    private Op aggregationOp;

    private GBHandler gbHandler;

    /**
     * Aggregate constructor
     * 
     * @param gbfield
     *            the 0-based index of the group-by field in the tuple, or
     *            NO_GROUPING if there is no grouping
     * @param gbfieldtype
     *            the type of the group by field (e.g., Type.INT_TYPE), or null
     *            if there is no grouping
     * @param afield
     *            the 0-based index of the aggregate field in the tuple
     * @param what
     *            the aggregation operator
     */

    public IntegerAggregator(int gbfield, Type gbfieldtype, int afield, Op what) {
        this.gbFieldIndex = gbfield;
        this.gbFieldType = gbfieldtype;
        this.aggregateFieldIndex = afield;
        this.aggregationOp = what;
        switch (what){
            case MIN:
                gbHandler =new MinHandler();
                break;
            case MAX:
                gbHandler =new MaxHandler();
                break;
            case AVG:
                gbHandler =new AvgHandler();
                break;
            case SUM:
                gbHandler =new SumHandler();
                break;
            case COUNT:
                gbHandler =new CountHandler();
                break;
            default:
                throw new UnsupportedOperationException("Unsupported aggregation operator ");
        }
    }

    /**
     * Merge a new tuple into the aggregate, grouping as indicated in the
     * constructor
     * 
     * @param tup
     *            the Tuple containing an aggregate field and a group-by field
     */
    public void mergeTupleIntoGroup(Tuple tup) {
        if(gbFieldType!=null&&(!tup.getField(gbFieldIndex).getType().equals(gbFieldType))){
            throw new IllegalArgumentException("Given tuple has wrong type");
        }
        String key;
        if (gbFieldIndex == NO_GROUPING) {
            key = NO_GROUPING_KEY;
        } else {
            key = tup.getField(gbFieldIndex).toString();
        }
        gbHandler.handle(key,tup.getField(aggregateFieldIndex));
    }
    /**
     * Create a OpIterator over group aggregate results.
     * 
     * @return a OpIterator whose tuples are the pair (groupVal, aggregateVal)
     *         if using group, or a single (aggregateVal) if no grouping. The
     *         aggregateVal is determined by the type of aggregate specified in
     *         the constructor.
     */
    public OpIterator iterator() {
        Map<String,Integer> results = gbHandler.getGbResult();
        Type[] types;
        String[] names;
        TupleDesc tupleDesc;
        List<Tuple> tuples = new ArrayList<>();
        if(gbFieldIndex==NO_GROUPING){
            types = new Type[]{Type.INT_TYPE};
            names = new String[]{"aggregateVal"};
            tupleDesc = new TupleDesc(types,names);
            for(Integer value:results.values()){
                Tuple tuple = new Tuple(tupleDesc);
                tuple.setField(0,new IntField(value));
                tuples.add(tuple);
            }
        }else{
            types = new Type[]{gbFieldType,Type.INT_TYPE};
            names = new String[]{"groupVal","aggregateVal"};
            tupleDesc = new TupleDesc(types,names);
            for(Map.Entry<String,Integer> entry:results.entrySet()){
                Tuple tuple = new Tuple(tupleDesc);
                if(gbFieldType==Type.INT_TYPE){
                    tuple.setField(0,new IntField(Integer.parseInt(entry.getKey())));
                }else{
                    tuple.setField(0,new StringField(entry.getKey(),entry.getKey().length()));
                }
                tuple.setField(1,new IntField(entry.getValue()));
                tuples.add(tuple);
            }
        }
        return new TupleIterator(tupleDesc,tuples);
    }

    private abstract class GBHandler{
        ConcurrentHashMap<String,Integer> gbResult;
        abstract void handle(String key,Field field);
        private GBHandler(){
            gbResult = new ConcurrentHashMap<>();
        }
        public Map<String,Integer> getGbResult(){
            return gbResult;
        }
    }
    private class CountHandler extends GBHandler {
        @Override
        public void handle(String key, Field field) {
            if(gbResult.containsKey(key)){
                gbResult.put(key,gbResult.get(key)+1);
            }else{
                gbResult.put(key,1);
            }
        }
    }
    private class SumHandler extends GBHandler{
        @Override
        public void handle(String key, Field field) {
            if(gbResult.containsKey(key)){
                gbResult.put(key,gbResult.get(key)+Integer.parseInt(field.toString()));
            }else{
                gbResult.put(key,Integer.parseInt(field.toString()));
            }
        }
    }
    private class MinHandler extends GBHandler{
        @Override
        void handle(String key, Field field) {
            int tmp = Integer.parseInt(field.toString());
            if(gbResult.containsKey(key)){
                int res = gbResult.get(key)<tmp?gbResult.get(key):tmp;
                gbResult.put(key, res);
            }else{
                gbResult.put(key,tmp);
            }
        }
    }
    private class MaxHandler extends GBHandler{
        @Override
        void handle(String key, Field field) {
            int tmp = Integer.parseInt(field.toString());
            if(gbResult.containsKey(key)){
                int res = gbResult.get(key)>tmp?gbResult.get(key):tmp;
                gbResult.put(key, res);
            }else{
                gbResult.put(key,tmp);
            }
        }
    }
    private class AvgHandler extends GBHandler{
        ConcurrentHashMap<String,Integer> sum;
        ConcurrentHashMap<String,Integer> count;
        private AvgHandler(){
            count = new ConcurrentHashMap<>();
            sum = new ConcurrentHashMap<>();
        }
        @Override
        public void handle(String key, Field field) {
            int tmp = Integer.parseInt(field.toString());
            if(gbResult.containsKey(key)){
                count.put(key,count.get(key)+1);
                sum.put(key,sum.get(key)+tmp);
            }else{
                count.put(key,1);
                sum.put(key,tmp);
            }
            gbResult.put(key,sum.get(key)/count.get(key));
        }
    }
}
```

StringAggregator类的实现和Integer类似，只需要实现count即可

Aggregate类只需要不断调用Aggregator提供的接口，加一层封装即可

### exercise3 HeapFile Mutability

> 这部分要求完成HeapPage和HeapFile中的插入、删除，主要就是更新bitmap中的信息

实现余下部分

* src/simpledb/HeapPage.java 

* src/simpledb/HeapFile.java 

	*(Note that you do not necessarily need to implement writePage at this point).*

* src/simpledb/BufferPool.java 中的

	* insertTuple() 
	* deleteTuple()

注意点：

1. heapFile的读页操作，都要应该从BufferPool里操作。比如加页
2. BufferPool里面的insertTuple需要做两件事，
	* 一是调用HeapFile的insertTuple
	* 二是把在脏页读入cache中，**一开始这里忘记读入cache，一直出错**。

### exercise4  Insert&Delete

> 调用exercise3中实现的方法即可

实现

* src/simpledb/Insert.java 
* src/simpledb/Delete.java

可以注意到，delete操作都不需要提供tableId，而insert操作需要提供tableId，这是因为delete的时候，可以从Tuple的RecordId中获取到tableId

### exercise5 page eviction

> 在BufferPool中实现页面淘汰策略，主要实现几个接口
>
> 1. `discardPage(PageId pid)`：用于移除cache中的页
> 2. `flushPage(PageId pid)`：如果是脏页，将脏页写入内存，并溢出脏位
> 3. `evictPage()`：自己实现一个页面淘汰策略，这里我实现的是随机淘汰
> 4. `flushAllPages()`：测试接口，无用
>
> 然后修改先前实现的BufferPool操作，保证cache中的页面不大于numPages

实现

* src/simpledb/BufferPool.java

这里我核心使用的是最简单的随机替换策略，这里其实**埋了一个优化点**，如果后面有机会可以优化为LRU或者LFU

```java
    /**
     * Discards a page from the buffer pool.
     * Flushes the page to disk to ensure dirty pages are updated on disk.
     */
    private synchronized  void evictPage() throws DbException {
        // 采用最简单的随机淘汰策略
        List<Integer> keys = new ArrayList<>(pages.keySet());
        int randomKey = keys.get(new Random().nextInt(keys.size()));
        PageId evictPid = pages.get(randomKey).getId();
        try {
            flushPage(evictPid);
        } catch (IOException e) {
            e.printStackTrace();
        }
        discardPage(evictPid);
    }
```

一个要注意的点就是，对于脏页若是要替换出去的话，得先刷到磁盘上再逐出内存BufferPool，不然会有数据不一致的风险

## reference

6.830 Lab 2: SimpleDB Operators：https://blog.csdn.net/hjw199666/article/details/103590963

MIT 6.830 Database System 数据库系统 Lab 2 实验报告：https://zhuanlan.zhihu.com/p/159186187