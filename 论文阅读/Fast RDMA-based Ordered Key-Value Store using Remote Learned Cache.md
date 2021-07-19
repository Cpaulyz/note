# Fast RDMA-based Ordered Key-Value Store using Remote Learned Cache

[TOC]

## 0 Background

### Traditional KVs use RPC (Server-centric)

![image-20210713152226150](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713152226150.png)

缺点：Sever CPU瓶颈

### Client-direct

![image-20210713152337953](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713152337953.png)

**NIC can read only one level per RTT!**

缺点：

* 支持 read操作
* 对于简单的index structure效果好，比如Hashtable，只需要O(1)次RTT
* 对于复杂的index structure效果一般，比如B+Tree，需要O(logn)次RTT

### Cache

![image-20210713152633317](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713152633317.png)

### Trade-off of existing KVs

![image-20210713152734268](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713152734268.png)

### overview of XSTORE

* 混合架构
	* server-centric update
	* learned cache for tree-based index
	* model retain in background thread
	* stale model handing

## 1 Introduction

四个主要贡献

* 引入 learned cache，使用ML模型来作为RDMA-bade、tree-backed KV store的indexed cache
* 混合架构：client-side(learned cache)+server-side(tree-based index)，实现动态负载（面对insert、delete等）
* 引入中间层
	1.  解耦
	2. 允许使用旧的（可能过期的）indexed cache来进行正确预测
* 一个原型，XStore

## 2 RDMA-based Key-Value Store

主要关注：基于网络（CS架构）的内存KV存储

* server端：KV、index存在主存中
* client端：通过给定接口进行请求，支持并发
	* get | update | scan | insert | delete

### 非RDMA设计

存在问题：CPU成为瓶颈

### RDMA设计

优点：减少CPU/内核负载

缺点：遍历复杂、需要多次网络传输

#### Server-centric design(S-RKV)

![image-20210707203413094](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707203413094.png)

优点：只需要两次传输，不管index结构是怎样的

缺点：CPU使用量高、成为性能瓶颈

#### Client-direct design(C-RKV)

![image-20210707203425178](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707203425178.png)

目的：减少服务端负载

优点：将CPU负载转移到客户端

缺点：

* 只适用于读多的场景，且只适用于只读请求（get、scan）；对于非只读的，比如update、insert、delete，还是需要服务端参与
* 需要很多次的的网络传输（logn）

### Index Caching

用于减少网络传输

缺点：

* 增加存储开销，如果cache整个树的话很大

## 3 Analysis of RDMA-based Ordered KVs

### CPU是S-RKV的瓶颈

![image-20210707210255291](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707210255291.png)

CPU很快成为server-centric design的瓶颈

### 遍历是C-RKV的最大困难

在C-RKV上进行遍历需要很多轮的网络传输(logN for tree-based index)，并且导致网络带宽饱和

![image-20210707210543765](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707210543765.png)

从图上来看，C-RKV的通量远小于S-RKV

### 树结构不是一个适合做index cache的结构

现存的index cache有的用相同的结构（tree）来作为缓存，作者认为这并不是一个好的设计，原因如下：

* tree-based index很大，遍历需要从root到leaf的随机访问

	尽管客户端可以通过只缓存上面几层来减少thrashing并最大化内存命中率。
	
	但这样依旧会造成unavoidable capacity misses。
	
	并且，如上面的图所示，影响最大的是缓存了几层，所以这只能说是在时间和空间上的一种折中。

* 遍历tree-based index是一个需要内存，但是计算量少的操作。本地缓存的话只是把远程访问变成本地访问罢了，仍然会引发*大量的内存随机访问和CPU cache miss*

	并且从测试结果来看，就算全缓存了，中间时延也没有S-RKV好
	
	![image-20210707212001482](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707212001482.png)

* 更新会传递性地造成缓存失效。

	更坏的是，越多树节点被缓存，性能就会被降低越多。因为保持一致性需要很复杂的检测机制，这会造成额外的负担。如图，仅仅因为一点点的插入，就会损失很多性能
	
	![image-20210707212229079](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707212229079.png)

## 4 Approach and Overview

### Opportunity: ML Models

![image-20210707233845122](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707233845122.png)

* CDF(累积分布函数)：映射 sorted key 到 sorted position of value
* 预测一个位置，并且有min_err和max_err，然后在这个区间内scan，找到最终的准确位置。其实这和B+树一样，只能定位到块上，具体在哪里还得在块中搜索
* 可以使用递归的模型

### Our approach: Learned Cache

可以克服传统方法的缺点，有以下的好处：

1. 传统的homogeneous structure只能缓存部分index，而ML Model可以以部分准确率为代价，缓存全部的index，进而防止capacity misses。而且，扫描只需要一次RDMA READ。此外，高效利用内存。
2. tree-based index的扫描要花费logN的时间，而ML Model会预测一个范围，从而减少因为fewer CPU cached和TLB miss引发的延迟。
3. 减少失效，update等操作只会降低准确率。Learned cache可以从network round trips和bandwidth usage的角度来保存失效cost

### Challenge: Dynamic Workloads

原来是怎么做的？

* 为插入的数据维护一个delta index（比如B+TREE），并定期与learned index进行合并
* 缺点
	* 获取delta index会增加额外的网络传输
	* 难以在客户端缓存delta index
	* 增加远程访问，并且会使得learned index失效

引出问题：how to make learned cache keep pace with dynamic workloads at low cost becomes a key challenge

### Overview of XStore

![image-20210707234818791](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210707234818791.png)

提出了一种在客户端使用learned cache，在服务端使用tree-based index的混合架构

* Server：XTree
* Client：XCache

使用client-direct design来处理读操作，使用server-centric design来处理增删改操作

GET(K)的流程如下

1. the client first predicts a range of positions for the key K using XCACHE and then fetches them using one RDMA READ. 
2. the client uses a local search to find the actual position and fetches the value using another RDMA READ.

INSERT(K,V)的流程如下

1. The server searches the lookup key K by traversing the B+tree index first and then inserts the new KV pair (K,V). 
2. XSTORE will partially retrain ML models for updated tree nodes in the background, and each client will individually fetch the models for XCACHE on demand.

## 5 Design and Implementation

### 5.1 Data Structure

* **XTree**

  ![image-20210708192928187](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708192928187.png)

  和传统的B树差不多，叶子节点的KV按照排序的

  - 24-bit incarnation
  - 8-bit counter
  - 32-bit right-link pointer to next sibling
  - keys with n slots and values with n slots
  	- separately but continuously 可能是为了能够在RDMA传输过程中，将keys和values分开进行传输

  > INCA我的理解的一种类似版本号的标记，在一些insert等操作中，节点会裂变，这时候INCA会改变。从客户端发来的请求，就会根据INCA来判断缓存是否是valid
  >
  > KV是分散但是连续存储的，这样做的好处可以参考Q&A中关于两次 RDMA READ 的回答

* **XCache**

	![image-20210708193317583](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708193317583.png)

	客户端有一个两层的Cache模型，还有一个 translation table(TT)

	* XModel：输入一个key，生成一个虚拟地址范围， (POS[..]) within a sorted array (logically stitching together all leaf nodes of XTREE)

	* TT：用于地址转换

		a valid bit, a 31-bit actual leaf-node number (ALN). 24-bit incarnation and 8-bit counter.

* **Training models and TT**

	![image-20210708193644273](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708193644273.png)

	server会在后台对model和TT进行(re-)train，客户都可以按需获取或者更新

	* 先训练 top model（line4），然后把K分配给 subs model（line9）

	* train each sub-model on a sorted array of its keys with a private logical position at a leaf node granularity (line 12-21) and calculate min- and max-error for every sub-model.

	* note that the keys in the leaf node across sub-models will be trained by both of sub-models (in several kset[i] due to the same predict of `xmodel.top.predict(k)×M)` 

		> 我理解这边的k是每层最底下的keys，这可以说明在16-18行处，又要把lnode拿出来，其实16行的for应该省略了很多东西，应该是对属于[start:end]中的每一个lnode进行遍历。
		>
		> 比如说，N=10，在lnode1中，k1,k2...k3被分到subs[0]，k4,k5,...,k10被分到了subs[1]，那在训练subs[0]和subs[1]的时候都会在16行for到lnode1，然后被“trained by both of sub-models ”

	* Each sub-model has independent logical positions and an own translation table, making it easy to retrain a sub-model individually when necessary.

* **A memory-performance trade-off**

	* model小，但是TT大

### 5.2 Client-direct Operations

* **GET**

  ![image-20210708195037148](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708195037148.png)

  1. 先预测哪个这个key在哪个sub model里面，然后拿取sub model，计算一波pos，除N以后就是lnodeID的预测start、end（line1-5）

  2. 在这个范围内遍历lnode，用TT作转换，根据 TT 表转译并通过 door bell batching 技术（该优化技术帮助我们用一次 RDMA READ 读取非连续的内存区域信息）读取到远程内存中叶子节点的信息，然后在本地进行 local search，计算所要 key 的实际地址。

  3. invalid in TT entry(line 9 and 17) would result in a fallback path, which **ships the get operation to the server and fetches updated models and TT entries using a single request**(i.e., server-centric design).

  	> Q:line9，有必要一个invalid就fallback吗？其他不再看看？
  	>
  	> A:有的，否则SCAN会出问题。但如果lookup不用于SCAN只用于GET的话，我觉得是可以不fallback，再看看其他entry的，只是最后不能说notfound，还得ship到server端看一次。

* **SCAN**

	看懂GET，SCAN基本上差不多

	1. 先用类似GET的方法，找到K的remote address
	2. 用TT表中的信息(provides CNT and ALN in sorted order by key)，预测剩下N个的位置
	3. use one RDMA READ with doorbell batching to fetch these leaf nodes

* **Non-existent Keys**

	![image-20210713205531267](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713205531267.png)

	如果要判断一个key不存在的话，预测的key需要可以将他包裹起来

	比如，key=6，先查LR0得到pos，然后判断出可能是在key={5，7}的node中，遍历node，才能说明没有key=2
	
	但如果是key=10，可能会出问题。首先需要明确的是，训练submodel的时候只会根据当前submodel的min_key和max_key进行训练，如图，即LR0只会根据1~7进行训练。这时候如果来一个key=10的查询，结果是不准确的，LR0会认为可能出现在key={17，18}的node中，这时候根本拿不到entry啊？？？
	
	* 解决方法：**Data augmentation**
	
		增加boundary key，也就是说，进行填充，比如将上图的key=10填入前一个存在key的pos
	
		> 与作者魏星达邮件联系后得知，这里论文中的 “we add a non-existent key in the gap (KEY=10) with the position of a previous KEY=4 into both sub-models (LR0 and LR1).”属于笔误
	
		但是我觉得可以有更好的办法，比如直接在submodel的数据结构里面加上范围不就可以了吗。。。

### 5.3 Server-centric Operations

* **Correctness**: no lost key，the reader must return a correct value for a given key, regardless of concurrent writers

* **Concurrency**: reuse HTM-based concurrent B+tree

* **Update**

	只改变value，没啥影响

	有一个优化点，*position hint*，可以在客户端预测index的位置，从而跳过服务端B-tree traversal的CPU开销，怎么说也能在update-heavy的情况下提升性能。这个优化改善了YCSB A(50&Update+50%Read)的性能。

* **Insert & Delete**

	![image-20210708202157507](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708202157507.png)

	XTREE chooses not to keep key-value pairs and reduces working set in the HTM region.

	* 删除：对于空的leaf node不会重复利用，避免影响模型训练
	* 插入：直接插入KV到slot中，如果leaf node满了，会进行分裂，分配一个新的node，原本node的KV会重新分配到这两个nodes中，并且改变INCA，使得clients可以意识到他分裂过了。(INCA的作用)

	* **Retraining and invalidation**

		The insert of a new leaf node will break the sorted(logical) order of leaf nodes and cause model retraining. 

		由于有TT来进行解耦，实际上可以允许旧的cache继续使用

		Server 端后台独立进行 sub-model 的训练以及新 TT 表的构建。

		错误会在INCA不一致的时候被发现，这时候请求会被ship到server中处理，XCache会从server处重新fetch新的模型和TT

	* **Optimization: speculative execution**

		A split of leaf node just moves the second half of key-value pairs (sorted by key) to its new sibling leaf node. Therefore, the prediction to the split node must still be mapped to this node or its new sibling. 到拆分节点的映射必须拆分到该节点（移除后半部分）或者它右侧的同级节点。

		如果用stale TT，发现INCA不一致了，The client will still find the look up key in the keys fetched from the split leaf node. If not found, the client will use its right-link pointer to fetch (the second half) keys from its sibling (one more RDMA READ).

		（个人觉得，如果INCA可以记录的话，可以稍微判断一下，如果是+1的话，那大概率可以用，如果差很多，那就算了，机会渺茫啊！）

	* **Model expansion**

		数据增大的话，只需要增加subs model就可以了（可能有一些阈值，比如数据量，max_err,min_err....），训练新的subs model，不会影响到server和client的正常运行，因为

		1. training models will not change or move data
		2. the top model can be trained over incomplete data
		3. the conflicting sub-model retraining can be made later
		4. client can use originally learned cache during model expansion

### 5.4 Durability

writes for persistence and failure recovery

Reuse the existing durability mechanism in the concurrent tree-based index extended by XTREE, like version numbers.

- XSTORE 应该将**写入（更新、插入和删除）记录**到存储在可靠存储中的日志文件，用于持久性和故障恢复（例如，服务器的本地磁盘）。 由于**基于 RDMA 的远程访问仅限于读取**（查找、获取和扫描），因此它们**不会涉及日志记录和恢复**。 此外，XMODEL 和 TT 与 XT REE（例如，虚拟地址）紧密相关。 因此，它们应该在恢复后重建。

### 5.5 Scaling out XStore

如果要使用集群的话，client maintains a separate learned cache for each server即可

各个server独立训练，需要注意：boundary keys should be added to the training set to cover the gap of non-existent keys between neighboring servers.

* 横向伸缩

- XSTORE 遵循**粗粒度方案** [57]，这是主要的解决方案，将**有序的键值存储分布在多个服务器上（横向扩展）**。 XSTORE 首先根据键的**基于范围的分区函数将键值对分配给服务器**。然后每个服务器为其分配的**键值对单独构建 XTREE**，并进一步训练**相应的 XMODEL 和 TT**。请注意，**应将边界键添加到训练集中**，以覆盖相邻服务器之间不存在的键的间隙。
- **客户端为每个服务器维护一个单独的学习缓存**，并使用相同的分区函数来决定**哪个服务器**应该执行给定的请求。
- 基于它，客户端可以执行 §5.2 和 §5.3 中提到的请求，但有一个例外——SCAN (K,N) 读取跨多个服务器的一系列键值对。在指定的服务器上**查找 K 之后**，客户端可能会发现预期的数字 (N) **超过了该服务器中剩余的键值对**。从下一个服务器上的第一个逻辑叶节点开始，客户端可以**预测包含其余键值对的叶节点**。最后，客户端对涉及的**每个服务器**使用一个 **RDMA READ** 来获取这些叶节点。

## 6 Discussion

## 7 Evaluation

> answer the following questions:
>
> 1. Compare to server-centric designs?
> 2. Compare to client-direct designs?
> 3. Dose XSTORE provide better trade-off?
>
> 一些小结论
>
> ![image-20210713153845494](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713153845494.png)

### 7.1 Experiment Setup

- **Testbed 环境**:

	- 15 client machines
	- 12-core Intel Xeon CPUs,
	- 128GB of RAM, and
	- 2 ConnectX-4 100Gbps IB RNICs
	- RNIC is used by threads on the same socket and connected to a Mellanox 100Gbps IB Switch.

- **Workloads**.

	- mainly focus on YCSB

		- A: update heavy
		- B: read mostly
		- C: read only
		- D: read latest
		- E: short ranges
		- F: read-modify-write

	- 100 million KV pairs initially (a 7-level tree-based index and a leaf level), 8-byte key and 8-byte value are used.

	- **Uniform and Zipfan key** distributions are evaluated

		- YCSB D: only Uniform and Latest key distributions

		> In a nutshell, the distribution affects how YCSB reads and scans over the keyspace:
		>
		> - `uniform`: each row has an equal probability to be read
		> - `zipfian`: some rows have more probability to be targeted by reads or scans. Those rows are called "hot set" or "hot spot" and represent popular data, for instance popular threads of a forum. You should set it up with: `hotspotdatafraction` and `hotspotopnfraction`. See `$YCSB_HOME/workloads/workload_template` for more details.

	![image-20210713140836182](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713140836182.png)

- **Compare targests**
	- DrTM-Tree
	- eRPC+Masstree (server-centric design) EMT
	- Cell (client-direct design)
	- RDMA-Memcached (RDMA version of mencached) RMC

### 7.2 YCSB Performance

![image-20210713141435603](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713141435603.png)

* **Read-only workload (YCSB C)**
	* XSORE can achieve 82 million requests perscond (**even a little higher than the optimal throughput**)
	* only uses one RDMA READ to fetch one leaf node per lookup.
	* The systems with server-centric design perform better due to better CPU cache locality
	* 18% drop in Zipfian distribution. suspec that
		- 多个 clients 读取小范围内存
		- 怀疑 RNIC 根据请求地址检查 RDMA 操作之间的冲突，即便没有冲突，这样的检查也会争夺 processing resources。
	
* **Static read-write workloads (YCSB A, B, and F).**
	* 什么叫static，就是没有插入新数据，而是update
	* update-heavy workloads (YCSB A)：XSTORE的CPU需要处理update，仍然是瓶颈。但是与其他的server-centric KVs相比，XSTORE的learned cache有优化
		* read-mostly workloads (YCSB B)：the read requests are less skewed interleaved with (10%) updates, compared to read-only workloads(YCSB C)
		* the server of XSTORE has not been saturated ; thus it is still sufficient to perform updates, compared to update-heavy workloads(YCSB A)
	* The performance of XSTORE on YCSB F is somewhere in between since it has about 75% reads.
	
* **Dynamic Workload (YCSB D and E)**

	* for DrTM-Tree and EMT: tree-based index
	* for Cell and XSTORE: cache invalidation
	* XSTORE overhead comes from two parts:
		* cache invalidations would increase RDMA operations due to fallbacks (RDMA-based RPC) and speculative execution (50% one more RDMA READ);
		* a dynamic dataset is always harder to learn than a static dataset due to the randomly inserted new keys

* **CPU utilizations of XSTORE**

	* XSTORE 使用两个辅助线程来维护XMODEL for dynamic workload （？？），导致服务端CPU使用率的提高。
	* 但相对server-centric的KVs而言（比如DrTM-TREE），CPU使用率还是小了很多

* **End-to-End latency**

	![image-20210713145007619](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713145007619.png)

	* when low load, server-centric KVs have lower latency, as one RPC round trip is faster than two one-sided RDMA operations(NIC_RPC compare to NIC_IDX and NIC_VAL)
	* 看不是很懂

### 7.3 Production Workload Performance

类似YCSB A，CPU是瓶颈

### 7.4 Scale-out Performance

* scale XSTORE by range-based partitioning a YCSB dataset with 600M keys into different numbers of RNICs.
* limited by the number of client machines

### 7.5 Model (Re-)Training and Expansion

![image-20210713150157698](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713150157698.png)

训练速度需要与动态负载的 insertion 速度匹配，模型 retraining 跟不上会影响整体的 throughput。

*For dynamic workloads, the throughput of XSTORE would decrease whe stale sub-models can not retrained in time*

图 14 对比说明了该问题。

可以设定随着 keys 数增多，达到一定阈值后，重新训练 sub-models 增加的新模型。Model Expansion （但是需要避免抖动。。。）

### 7.6 Memory Footprint of XCache

* TT：依赖于leaf node，而不是interval node
* 占用小，因为只需要保存XMODEL

## Future work

![image-20210713154240829](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210713154240829.png)

## Some Q&A

### 关于两次 RDMA READ

> 问题名称：XTree 叶子节点的 Value 保存的是指具体的数据，还是指向数据部分的地址。
>
> XSTORE 在 value 时定长的时候保存的是具体的数据，不定长的时候储存的是地址。
>
> 这里我疑惑如果保存的是具体数据，我们为什么还需要在这里确定 key 的“实际地址”后再进行一次 RDMA READ，value 值不是可以再叶子节点中读取到吗？
>
> 你的问题是正确的。理论来说，如果叶子存的是具体数据，那么一次 RDMA 就可以完全读回。这边 XSTORE 使用两次是为了节约 RDMA 读的 payload。
>
> 举例来说，如果使用一次 RDMA，则需要读取叶子里的所有数据加上 Key，这样会读取过多的值 $(n *sizeof(Key) + n* sizeof(Value)其中,其中n$ 是叶子节点最多的 item 数量），造成 RDMA bandwidth 的浪费。
>
> 如果使用两次 RDMA，则总共需要读 n∗sizeof(Key)+sizeof(Value) 数据，如果 Value 的大小比 Key 大很多（通常是这种情况），则会更加高效。

### 硬件事务内存（Hardware Transactional Memory, HTM）

> 硬件事务内存系统一般是在每个处理器核心中增加事务cache，同时在CPU指令集中增加事务相关的指令。在执行事务的过程中，事务访问过的数据会缓存在事务cache中，修改过的cache一致性协议会检测各事务之间的冲突。如果产生冲突，对应的事务cache中修改过的数据全部作废，该事务重新执行；如果没有冲突产生，就在事务结束时把事务cache中的数据更新到内存中。
> 硬件事务内存系统完全由硬件电路实现，所以其性能相较于软件事务内存系统有很大的优势，但是由于硬件资源的限制，所以硬件事务内存无法处理大型事务（在处理大事务的过程中会产生大量的共享数据修改操作，这些修改记录的大小超过处理器内事务cache的大小）。为了保证事务的完全执行，现有的商用硬件事务内存系统会加入串行部分，也就是“锁”，对于一部分硬件事务内存实在无法完成的事务就用串行部分完成，这虽然损失了部分性能，但至少保证了功能的完整性。
> 在具体实现时有2个要点： 1. 使用处理器内部的事务cache来存放事务执行过程中更新的数据，这一类的经典代表：TCC 2. 使用cache一致性协议来检测事务之间共享数据访问的冲突，这一类的经典代表：LogTM.
> from https://zhuanlan.zhihu.com/p/151425608

### 什么是RDMA？

https://blog.csdn.net/qq_40323844/article/details/90680159

### 为什么不用hash而要用BTree？

* 空间利用率低
* 对于kv存储引擎的索引，根据key是否有序，分为两种：一种是key如果无序，则可以采用hash的方法；另一种是，如果key有序，则可以采用B+树的方法，或者类B+树的方法。
* BTree可以大大利用CPU的缓存，减少随机内存访问

## reference

https://edwardzcn.github.io/post/7a8a66e3.html

https://www.grayxu.cn/2020/11/20/XStore/

https://blog.shunzi.tech/post/osdi-xstore/

https://dev.tube/video/Qv-0YL_SII4



# 相关阅读

## 关于随机内存访问

https://zhuanlan.zhihu.com/p/86513504

## 关于 Learned Index 技术

> SIGMOD18 The Case for Learned Index Structures
>
> [浅谈Lenared Index Structure及相关的坑](https://zhuanlan.zhihu.com/p/52569266)
>
> [学习索引(learned index)技术](https://zhuanlan.zhihu.com/p/222773618)
>
> [机器学习思路优化B Tree](https://zhuanlan.zhihu.com/p/67021855)

### 想法

索引的过程本身就是模型：

* Range Index索引可以看做是从给定key到一个排序数组position的预测过程
* Point Index索引可以看做是从给定key到一个未排序数组的position的预测过程
* Existence Index索引可以看做是预测一个给定key是否存在（0或1）

### ML模型可能存在的问题

* **Semantic Guarantees**. 通过数据训练得到机器学习的模型是存在误差的，但是我们设计的索引经常需要满足严格的限制条件。举例来说，bloom-filter的优势之一是可能会有把不存在的错判成存在（False Positive）但绝不会把存在的错判成不存在（False Negative）。
* **Overfitting**. 如果避免模型的overfitting？换句话说，如果当前训练的模型只能memorize当前数据形态而并不能generalize新的数据，该怎么办？
* **No Specialization**. 如果数据并没有任何特殊性，如完全随机或者不能被我们的设计模型架构所归纳，该如何处理？另一种情况是，新的数据破坏了之前模型归纳出的特殊性，比如新的数据来源于一个新的或者是变化了的distribution。
* **Evaluate Efficiency**. 模型的inference过程相比显式索引数据结构的计算更昂贵。

### Range Index

主要是关于 Range Index 的部分

提出使用机器学习模型代替传统的B树索引，并在真实数据集上取得了不错的效果，但其提出的模型假设工作负载是静态的、只读的，对于索引更新问题没有提出很好的解决办法. 

![image-20210708165718340](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210708165718340.png)

* 首先B-Tree 如图所示，输入是一个key，输出是对应要查询record的position的区间  `[pos, pos+pageSize]`
* Learned Index的角度来看，输入一个Key，根据训练完的模型，输出要查询的record的position区间`[pos-min_err, pos+max_err]`

### CDF

> 怎么把 Range Index 模型抽象为 CDF，个人认为这是最难理解的部分

Range index实际上是描述一个数据点到一个有序数组的函数关系，也就是说这个函数关系"模拟"出了数据的累计分布函数（CDF）。

因此，可以这么来定义这个函数 ` p = CDF(key)* N `

其中，p是position，CDF(key)是`Prob(X<=key)`，N是所有key的总数

拟合数据的CDF函数是一个active的研究方向，可以受益于已有的研究成果。