## DAG生成和stage的划分

源码解析：https://www.cnblogs.com/johnny666888/p/11233982.html

* 一个个RDD批量执行 (×)
* Spark将颗粒度划分到task，从后往前，逐条执行。 (√)

划分算法：从后往前，遇到Shuffle就断开，遇到窄依赖就加入，每个 stage 里面 task 的数目由该 stage 最后一个 RDD 中的 partition 个数决定。

![image-20200925104937099](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20200925104937099.png)

> https://www.jianshu.com/p/91ec30f507e8

### [ResultStage和ShuffleMapStage](https://www.cnblogs.com/laogou-idea/p/12488794.html)

首先 job 的个数取决于 active 行动算子的个数。当流程执行一个 active 行动算子，spark就会生成一个 job 。

而一个 job 分为多个 stage 阶段，stage 的个数取决于宽依赖的个数，对于宽依赖大家可以自行百度，我这里说一个简单的概念。即分区内的数据如果改变（打乱重新组合）了就算是宽依赖。宽依赖一定伴随着shuffle。

其次一个 stage 阶段包含多个 task 任务，task 任务的个数取决于当前stage阶段的最后一个rdd 的分区数。不同的rdd的分区数计算规则也不同。要根据具体的生产环境确定。



ShuffleMapstage主要记录的是为shuffle阶段产生数据的前的一个状态。那它为什么要记录这个呢？

因为一旦进行shuffle阶段之后，必然会产生宽依赖。产生宽依赖就意味着这个stage阶段的结束。也就是说ShuffleMapStage类就是这个stage阶段结束的最后一个rdd时的状态。

那为什么要保留这个状态呢。

因为上面说过task的数量取决于stage阶段最后一个rdd的分区数。所以当spark想要确定task的数量的时候他就会通过ShuffleMapStage来获取。

其实这样说不是很严谨。因为stage阶段的结束不止只有宽依赖这一种情况。还有一种情况就是整个程序运行行动算子的时候。因为运行完行动算子，此次job就会结束，最后一个stage阶段的task数量只能取决于运行此次行动算子之前的rdd的分区数。此时也就有了类ResultStage

> [白话spark](https://blog.csdn.net/handoking/article/details/81122877)
>
> 