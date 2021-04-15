# LAB3

* 实现TableStats类中的方法，使得可以估计过滤器的选择率和遍历的代价，使用直方图或者你发明的其他方式展示结果。
* 实现JoinOptimizer类中的方法，使得可以估计cost和join的选择率
* 实现JoinOptimizer类中的orderJoins方法。这个方法会根据前两步计算出的数据，生成一个最佳的joins顺序

## CBO(cost-based optimizer)

### RBO & CBO

> SQL优化的发展，则可以分为两个阶段，即RBO（Rule Base Optimization），和CBO（Cost Base Optimization）。

RBO，RBO主要是开发人员在使用SQL的过程中，有些发现有些通用的规则，可以显著提高SQL执行的效率

例如，我们都知道join是非常耗时的一个操作，且性能与join双方数据量大小呈线性关系（通常情况下）。那么很自然的一个优化，就是尽可能减少join左右双方的数据量，于是就想到了先filter再join这样一个rule。而非常多个类似的rule，就构成了RBO。

![image-20210412202812737](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210412202812737.png)

### main idea of CBO

* 利用关于表的统计数据，来估计不同查询计划的cost。通常来说，cost与**join、selection的基数**、**filter的选择率**和**join的谓词有关**。
* 使用数据来对join和select进行排序，并选择最佳的实现方式



## LAB3

### exercise1

难点在于区间怎么定义，边界条件要小心，必要时可以特判

注意width计算，要从Bucket里获得，而不是直接拿widths 整个bug在exercise2才发现

### exercise2

使用Histogram即可

### exercise3

计算join的cost和cardinality，也就是join操作的开销和join后的基数预估

* estimateJoinCost(LogicalJoinNode j, int card1, int card2, double cost1, double cost2)

	* ```
		joincost(t1 join t2) = scancost(t1) + ntups(t1) x scancost(t2) //IO cost
		+ ntups(t1) x ntups(t2) //CPU cost
		```

* estimateJoinCardinality(LogicalJoinNode j, int card1, int card2, boolean t1pkey, boolean t2pkey)

	* 对于equality joins

		当一个属性是primary key，由表连接产生的tuples数量不能大于non-primary key属性的选择数。

		对于没有primary key的equality joins，很难说连接输出的大小是多少，可以是两表被选择数的乘积（如果两表的所有tuples都有相同的值），或者也可以是0。

	* 对于范围scans，很难说清楚明确的数量。输出的数量应该与输入的数量是成比例的，可以预估一个固定的分数代表range scans产生的向量叉积（cross-product），比如30%。总的来说，range join的开销应该大于相同大小两表的non-primary key equality join开销。

基本按照框架走就可

## reference

https://zhuanlan.zhihu.com/p/248796415