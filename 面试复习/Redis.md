# Redis

[TOC]



### 缓存数据的处理流程

和cache一样

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210309162326395.png" alt="image-20210309162326395" style="zoom:67%;" />

###  为什么需要redis作缓存

从高性能和高并发来说

* **高性能**

	访问内存的速度远大于访问磁盘的速度

	假如用户第一次访问数据库中的某些数据的话，这个过程是比较慢，毕竟是从硬盘中读取的。但是，如果说，用户访问的数据属于高频数据并且不会经常改变的话，那么我们就可以很放心地将该用户访问的数据存在缓存中。

	那就是保证用户下一次再访问这些数据的时候就可以直接从缓存中获取了。操作缓存就是直接操作内存，所以速度相当快。

* **高并发**

	一般像 MySQL 这类的数据库的 QPS 大概都在 1w 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（就单机 redis 的情况，redis 集群的话会更高）

> QPS（Query Per Second）：服务器每秒可以执行的查询次数；

 ### redis如何判断数据是否过期

Redis 通过一个叫做过期字典（可以看作是hash表）来保存数据过期的时间。过期字典的键指向Redis数据库中的某个key(键)，过期字典的值是一个long long类型的整数，这个整数保存了key所指向的数据库键的过期时间（毫秒精度的UNIX时间戳）。

![image-20210309163056114](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210309163056114.png)

### 过期数据删除策略？

常用的过期数据的删除策略就两个（重要！自己造缓存轮子的时候需要格外考虑的东西）：

1. **惰性删除** ：只会在取出key的时候才对数据进行过期检查。这样对CPU最友好，但是可能会造成太多过期 key 没有被删除。
2. **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期key操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。

定期删除对内存更加友好，惰性删除对CPU更加友好。两者各有千秋，所以Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就Out of memory了。

怎么解决这个问题呢？答案就是： **Redis 内存淘汰机制。**

### 内存淘汰机制

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中LRU
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中LRU
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

4.0 版本后增加以下两种：

1. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
2. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key

### 持久化技术，具体讲讲哪种比较好

[Redis(三)：持久化RDB与AOF详解#](https://www.cnblogs.com/cpaulyz/p/14480402.html#redis三：持久化rdb与aof详解)

### 事务

[Redis(四)：事务#](https://www.cnblogs.com/cpaulyz/p/14506464.html#redis四：事务)

### 缓存穿透

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。举个例子：某个黑客故意制造我们缓存中不存在的 key 发起大量请求，导致大量请求落到数据库。

* **解决方法1：缓存无效key**

	如果缓存和数据库都查不到某个 key 的数据就写一个到 Redis 中去并设置过期时间，具体命令如下： `SET key value EX 10086` 。这种方式可以解决请求的 key 变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效的 key 的过期时间设置短一点比如 1 分钟。

* **解决方法2：布隆过滤器**

	**布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**因为多个值可能hash到同一个hashcode上

	https://www.cnblogs.com/ysocean/p/12594982.html

	本质上是bitmap，可用选择多个hash函数

### 缓存雪崩

#### 原因

* **缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求。** 这就好比雪崩一样，摧枯拉朽之势，数据库的压力可想而知，可能直接就被这么多请求弄宕机了。

	举个例子：系统的缓存模块出了问题比如宕机导致不可用。造成系统的所有访问，都要走数据库。

* **有一些被大量访问数据（热点缓存）在某一时刻大面积失效，导致对应的请求直接落到了数据库上。** 

	举个例子 ：秒杀开始 12 个小时之前，我们统一存放了一批商品到 Redis 中，设置的缓存过期时间也是 12 个小时，那么秒杀开始的时候，这些秒杀的商品的访问直接就失效了。导致的情况就是，相应的请求直接就落到了数据库上，就像雪崩一样可怕。

#### 解决

**针对 Redis 服务不可用的情况：**

1. 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用。
2. 限流，避免同时处理大量的请求。

**针对热点缓存失效的情况：**

1. 设置不同的失效时间比如随机设置缓存的失效时间。
2. 缓存永不失效。

**其他方案**

1. Redis缓存预热，避免冷启动
2. 通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
3. 时间T到达后，cache中的key和value不会被清掉，而只是被标记为过期（逻辑上过期，物理上不过期），然后程序异步去刷新cache。而后续部分读线程在前面的线程刷新cache成功之前，暂时获取cache中旧的value返回。一旦cache刷新成功，后续所有线程就能直接获取cache中新的value。可以看到，这个思路很大程度上减少了排斥锁的使用（虽然并没有完全消除排斥锁）



### 缓存和数据库一致性问题

https://www.cnblogs.com/cpaulyz/p/14566951.html

### 五种数据结构

### 跳表，为什么MySQL不用跳表作为索引？为什么redis不用B+树或者红黑树？

https://blog.csdn.net/Linuxhus/article/details/113651182

空间复杂度O(n)

查找的时间复杂度O(logn)

类似索引的思想

因为MySQL是不是内存数据库，数据是存在磁盘上的，每次读一个B+树节点的话相当于读一个页，一般查找一个数据只需要IO三次

redis不用红黑树是因为无法做到范围查询

redis不用B+树是可能是因为设计复杂，奥卡姆剃刀，而且跳表和底层的双向链表设计挺一致的，很有美感

### 单线程模型，IO多路复用

为什么不用多线程？因为瓶颈在网络传输

https://blog.csdn.net/diweikang/article/details/90346020

### redis对象

```c
struct RedisObject { 
    int4 type; // 4bits  类型
    int4 encoding; // 4bits 存储格式
    int24 lru; // 24bits 记录LRU信息
    int32 refcount; // 4bytes 
    void *ptr; // 8bytes，64-bit system 
} robj;
```

不同的对象具有不同的类型 type(4bit)，同一个类型的 type 会有不同的存储形式 encoding(4bit)。

为了记录对象的 LRU 信息，使用了 24 个 bit 的 lru 来记录 LRU 信息。

每个对象都有个引用计数 refcount，当引用计数为零时，对象就会被销毁，内存被回收。ptr 指针将指向对象内容 (body) 的具体存储位置。

一个 RedisObject 对象头共需要占据 16 字节的存储空间。

### 字符串SDS

https://blog.csdn.net/qq193423571/article/details/81637075

底层实现是简单动态字符串**sds**(simple dynamic string)，是可以修改的字符串。

它类似于Java中的ArrayList，它采用**预分配冗余空间**的方式来减少内存的频繁分配

**SDS本质上就是char \***，因为有了表头**sdshdr**结构的存在，所以SDS比传统C字符串在某些方面更加优秀，并且能够兼容传统C字符串。

sds在Redis中是实现字符串对象的工具，并且完全取代char*..sds是二进制安全的，它可以存储任意二进制数据，不像C语言字符串那样以‘\0’来标识字符串结束，

因为传统C字符串符合ASCII编码，这种编码的操作的特点就是：遇零则止 。即，当读一个字符串时，只要遇到’\0’结尾，就认为到达末尾，就忽略’\0’结尾以后的所有字符。因此，如果传统字符串保存图片，视频等二进制文件，操作文件时就被截断了。

SDS表头的buf被定义为字节数组，因为判断是否到达字符串结尾的依据则是表头的len成员，这意味着它可以存放任何二进制的数据和文本数据，包括’\0’

SDS 和传统的 C 字符串获得的做法不同，传统的C字符串遍历字符串的长度，遇零则止，复杂度为O(n)。而SDS表头的len成员就保存着字符串长度，所以获得字符串长度的操作复杂度为O(1)。

总结下sds的特点是：**可动态扩展内存、二进制安全、快速遍历字符串 和与传统的C语言字符串类型兼容。**

Redis 的字符串共有两种存储方式，在长度特别短时，使用 emb 形式存储 (**embedded**)，当长度超过 **44** 时，使用 **raw** 形式存储。 

![image-20210322105555039](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210322105555039.png)

embstr 存储形式是这样一种存储形式，它将 RedisObject 对象头和 SDS 对象连续存在一起，使用 malloc 方法一次分配。而 raw 存储形式不一样，它需要两次 malloc，两个对象头在内存地址上一般是不连续的。

在字符串比较小时，SDS 对象头的大小是capacity+3——SDS结构体的内存大小至少是 3。意味着分配一个字符串的最小空间占用为 19 字节 (16+3)。

如果总体超出了 64 字节，Redis 认为它是一个大字符串，不再使用 emdstr 形式存储，而该用 raw 形式。而64-19-结尾的\0，所以empstr只能容纳44字节。

**扩容**：扩容策略是字符串在长度小于 SDS_MAX_PREALLOC 之前，扩容空间采用加倍策略，也就是保留 100% 的冗余空间。当长度超过 SDS_MAX_PREALLOC 之后，为了避免加倍后的冗余空间过大而导致浪费，每次扩容只会多分配 SDS_MAX_PREALLOC大小的冗余空间。

### 双向链表

![image-20210322105800360](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210322105800360.png)

### dict

https://blog.csdn.net/cxc576502021/article/details/82974940

https://blog.csdn.net/cqk0100/article/details/80400811

#### 数据结构

```c
/* 哈希表节点 */
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;

/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
/* 哈希表
 * 每个字典都使用两个哈希表，以实现渐进式 rehash 。
 */
typedef struct dictht {
    // 哈希表数组
    // 可以看作是：一个哈希表数组，数组的每个项是entry链表的头结点（链地址法解决哈希冲突）
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;
/* 字典 */
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */
} dict;
```

#### rehash

1. 为dict的哈希表ht[1]分配空间，分配的空间大小取决于操作类型和当前键值对数量ht[0].used
2. 重新计算ht[0]中所有键的哈希值和索引值，将相应的键值对迁移到ht[1]的指定位置中去，需要注意的是，这个过程是渐进式完成的，不然如果字典很大的话全部迁移完需要一定的时间，这段时间内Redis服务器就不可用了哟
3. 当ht[0]的所有键值对都迁移到ht[1]中去后（此时ht[0]会变成空表），把ht[1]设置为ht[0]，并重新在ht[1]上新建一个空表，为下次rehash做准备

上面说到字典的哈希表rehash的过程是渐进式的，这里渐进的过程具体如下：

1. 为ht[1]分配空间，此时字典同时持有ht[0]和ht[1]
2. 将rehashidx设为0，表示rehash正式开始
3. 在rehash期间，每次对字典执行任意操作时，程序除了执行对应操作之外，还会顺带将ht[0]在rehashidx索引上的所有键值对rehash到ht[1]，操作完后将rehashidx的值加一
4. 在rehash期间，对字典进行ht[0].size次操作之后，rehashidx的值会增加到ht[0].size，此时ht[0]的所有键值对都已经迁移到ht[1]了，程序会将rehashidx重新置为-1，以此表示rehash完成

`int dictRehash(dict *d, int n) {}`，N是rehash的桶数

**渐进式哈希的精髓在于**：数据的迁移不是一次性完成的，而是可以通过dictRehash()这个函数分步规划的，并且调用方可以及时知道是否需要继续进行渐进式哈希操作。如果dict数据结构中存储了海量的数据，那么一次性迁移势必带来redis性能的下降，别忘了redis是单线程模型，在实时性要求高的场景下这可能是致命的。而渐进式哈希则将这种代价可控地分摊了，调用方可以在dict做插入，删除，更新的时候执行dictRehash()，最小化数据迁移的代价。
在迁移的过程中，数据是在新表还是旧表中并不是一个非常急迫的需求，迁移的过程并不会丢失数据，在旧表中找不到再到新表中寻找就是了。

这里需要注意的是，在rehash的过程中，ht[0]和ht[1]可能同时存在键值对，因此在执行查询操作的时候两个哈希表都得查，而如果是执行插入键值对操作，则直接在ht[1]上操作就行。


最后说下Redis在什么条件下会对哈希表进行扩展或收缩：

1. 服务器当前没有在执行BGSAVE或BGREWRITEAOF命令且哈希表的负载因子大于等于1时进行扩展操作

2. 服务器正在执行BGSAVE或BGREWRITEAOF命令且哈希表的负载因子大于等于5时进行扩展操作

	（这里负载因子的计算公式为：负载因子=哈希表当前保存节点数/哈希表大小，而之所以在服务器进行BGSAVE或BGREWRITEAOF的时候负载因子比较大才进行扩展操作是因为此时Redis会创建子进程，而大多数操作系统采取了写时复制的技术来优化子进程使用效率，不适合在这种时候会做大规模的数据迁移活动，说白了就是为了节约内存和提升效率）

3. 当前负载因子小于0.1时进行收缩操作
  

### zset

dict+跳表

插入操作：https://blog.csdn.net/wanghao112956/article/details/102544228

```c
//    从最高索引层开始遍历，根据 score 找到它的前驱节点，用 update 数组进行保存
//    每一层得进行节点的插入，并计算更新 span 值
//    修改 backward 指针与 tail 指针
zskiplistNode *zslInsert(zskiplist *zsl, double score, sds ele) {
    //update数组将用于记录新节点在每一层索引的目标插入位置
    zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
    //rank数组记录目标节点每一层的排名
    unsigned int rank[ZSKIPLIST_MAXLEVEL];
    int i, level;

    serverAssert(!isnan(score));
    //指向哨兵节点
    x = zsl->header;
    //这一段就是遍历每一层索引，找到最后一个小于当前给定score值的节点
    //从高层索引向底层索引遍历
    for (i = zsl->level-1; i >= 0; i--) {
        //rank记录的是节点的排名，正常情况下给它初始值等于上一层目标节点的排名
        //如果当前正在遍历最高层索引，那么这个初始值暂时给0
        rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
        while (x->level[i].forward &&
                (x->level[i].forward->score < score ||
                    (x->level[i].forward->score == score &&
                    sdscmp(x->level[i].forward->ele,ele) < 0)))
        {
            //我们说过level结构中，span表示的是与后面一个节点的跨度
            //rank[i]最终会得到我们要找的目标节点的排名，也就是它前面有多少个节点
            rank[i] += x->level[i].span;
            //挪动指针
            x = x->level[i].forward;
        }
        update[i] = x;
    }
    //至此，update数组中已经记录好，每一层最后一个小于给定score值的节点
    //我们的新节点只需要插在他们后即可
    
    //random算法获取一个平衡跳表的level值，标志着我们的新节点将要在哪些索引出现
    //具体算法这里不做分析，你也可以私下找我讨论
    level = zslRandomLevel();
    //如果产生值大于当前跳表最高索引
    if (level > zsl->level) {
        //为高出来的索引层赋初始值，update[i]指向哨兵节点
        for (i = zsl->level; i < level; i++) {
            rank[i] = 0;
            update[i] = zsl->header;
            update[i]->level[i].span = zsl->length;
        }
        zsl->level = level;
    }
    //根据score和ele创建节点
    x = zslCreateNode(level,score,ele);
    //每一索引层得进行新节点插入，建议对照我之前给出的跳表示意图
    for (i = 0; i < level; i++) {
        //断开指针，插入新节点
        x->level[i].forward = update[i]->level[i].forward;
        update[i]->level[i].forward = x;

        //rank[0]等于新节点再最底层链表的排名，就是它前面有多少个节点
        //update[i]->level[i].span记录的是目标节点与后一个索引节点之间的跨度，即跨越了多少个节点
        //得到新插入节点与后一个索引节点之间的跨度
        x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
        //修改目标节点的span值
        update[i]->level[i].span = (rank[0] - rank[i]) + 1;
    }

    //如果上面产生的平衡level大于跳表最高使用索引，我们上面说会为高出部分做初始化
    //这里是自增他们的span值，因为新插入了一个节点，跨度自然要增加
    for (i = level; i < zsl->level; i++) {
        update[i]->level[i].span++;
    }

    //修改 backward 指针与 tail 指针
    x->backward = (update[0] == zsl->header) ? NULL : update[0];
    if (x->level[0].forward)
        x->level[0].forward->backward = x;
    else
        zsl->tail = x;
    zsl->length++;
    return x;
}
```

