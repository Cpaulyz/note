# Redis

## Redis介绍

### 什么是redis

**RE**mote **DI**ctionary **S**erver 远程字典服务器

Redis 与其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储
- Redis支持数据的备份，即master-slave模式的数据备份

> KV+cache+persistenc

### 能干什么

- 内存存储和持久化：redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务
- 取最新N个数据的操作，如：可以将最新的10条评论的ID放在Redis的List集合里面
- 模拟类似于HttpSession这种需要设定过期时间的功能
- 发布、订阅消息系统
- 定时器、计数器

## HelloWorld

### 安装

```shell
wget http://download.redis.io/releases/redis-6.0.6.tar.gz
tar -zxvf redis-6.0.6.tar.gz 
cd redis-6.0.6
make
make install
```

这里踩了个坑，安装的时候出现了一堆类似这样的错误

![image-20210218171844824](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218171844824.png)

解决方法：升级GCC

```
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

之后重新make就可以安装成功了

![image-20210218172739704](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218172739704.png)

### hello world

修改redis文件夹下的配置文件`redis.conf`

![image-20210218173510336](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218173510336.png)

启动redis-server，`/usr/local/bin/redis-server /opt/redis-6.0.6/redis.conf`

启动客户端，`/usr/local/bin/redis-cli -p 6379`

![image-20210218173820374](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218173820374.png)

### redis启动后的杂项知识

* 性能测试：启动服务后，运行`/usr/local/bin/redis-benchmark`

* 单进程   

	- 单进程模型来处理客户端的请求。对读写等事件的响应 是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率
	- Epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本， 它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。

* 默认16个数据库，类似数组下表从零开始，初始默认使用零号库，可在配置文件配置

	![image-20210218174418018](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218174418018.png)

* `select`命令切换数据库

	* 不同库的情况：

		![image-20210218174553101](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218174553101.png)

* `dbsize`查看当前数据库的key的数量

	![image-20210218174659146](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218174659146.png)

* `flushdb`：清空当前库

* `flushall`；通杀全部库

* 统一密码管理，16个库都是同样密码，要么都OK要么一个也连接不上

* Redis索引都是从零开始

* 默认端口是6379

## 数据类型

### 五大数据类型

- String（字符串）   
	- string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
	- string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
	- string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M
- Hash（哈希，类似java里的Map）   
	- Redis hash 是一个键值对集合。
	- Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
	- 类似Java里面的Map<String,Object>
- List（列表）   
	- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。
	- 它的底层实际是个链表
- Set（集合）   
	- Redis的Set是string类型的**无序**集合。它是通过HashTable实现实现的
- Zset(sorted set：有序集合)   
	- Redis zset 和 set 一样也是string类型元素的集合，且不允许重复的成员。
	- 不同的是每个元素都会关联一个double类型的分数。
	- redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数(score)却可以重复。
- 哪里去获得redis常见数据类型操作命令   
	- [Redis 命令参考](http://redisdoc.com/)
	- [Redis 官网命令参考](https://redis.io/commands)

### key关键字

#### 常用

| 命令                                      | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| DEL key                                   | 该命令用于在 key 存在时删除 key。                            |
| DUMP key                                  | 序列化给定 key ，并返回被序列化的值。                        |
| EXISTS key                                | 检查给定 key 是否存在。                                      |
| EXPIRE key seconds                        | 为给定 key 设置过期时间，以秒计。                            |
| EXPIREAT key timestamp                    | EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| PEXPIRE key milliseconds                  | 设置 key 的过期时间以毫秒计。                                |
| PEXPIREAT key milliseconds-timestamp      | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计           |
| KEYS pattern                              | 查找所有符合给定模式( pattern)的 key 。                      |
| MOVE key db                               | 将当前数据库的 key 移动到给定的数据库 db 当中。              |
| PERSIST key                               | 移除 key 的过期时间，key 将持久保持。                        |
| PTTL key                                  | 以毫秒为单位返回 key 的剩余的过期时间。                      |
| TTL key                                   | 以秒为单位，返回给定 key 的剩余生存时间(TTL, **time to live**)。 |
| RANDOMKEY                                 | 从当前数据库中随机返回一个 key 。                            |
| RENAME key newkey                         | 修改 key 的名称                                              |
| RENAMENX key newkey                       | 仅当 newkey 不存在时，将 key 改名为 newkey 。                |
| SCAN cursor [MATCH pattern] [COUNT count] | 迭代数据库中的数据库键。                                     |
| TYPE key                                  | 返回 key 所储存的值的类型。                                  |

#### 案例

- keys \*
- exists key的名字，判断某个key是否存在
- move key db —>当前库就没有了，被移除了
- expire key 秒钟：为给定的key设置过期时间
- ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期
- type key 查看你的key是什么类型
- set key value 设置key，如果已经存在会**覆盖**

### String

单值单value

#### 常用

| 命令                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| SET key value                  | 设置指定 key 的值                                            |
| GET key                        | 获取指定 key 的值。                                          |
| GETRANGE key start end         | 返回 key 中字符串值的子字符                                  |
| GETSET key value               | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。   |
| GETBIT key offset              | 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。         |
| MGET key1 [key2…]              | 获取所有(一个或多个)给定 key 的值。                          |
| SETBIT key offset value        | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。   |
| SETEX key seconds value        | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。 |
| SETNX key value                | 只有在 key 不存在时设置 key 的值。                           |
| SETRANGE key offset value      | 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| STRLEN key                     | 返回 key 所储存的字符串值的长度。                            |
| MSET key value [key value …]   | 同时设置一个或多个 key-value 对。                            |
| MSETNX key value [key value …] | 同时设置一个或多个 key-value 对，当且仅当**所有**给定 key 都不存在。 |
| PSETEX key milliseconds value  | 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。 |
| INCR key                       | 将 key 中储存的数字值增一。                                  |
| INCRBY key increment           | 将 key 所储存的值加上给定的增量值（increment） 。            |
| INCRBYFLOAT key increment      | 将 key 所储存的值加上给定的浮点增量值（increment） 。        |
| DECR key                       | 将 key 中储存的数字值减一。                                  |
| DECRBY key decrement           | key 所储存的值减去给定的减量值（decrement） 。               |
| APPEND key value               | 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。 |

#### 案例

- set/get/del/append/strlen

- Incr/decr/incrby/decrby,一定要是数字才能进行加减

- getrange/setrange

	![image-20210218182815271](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210218182815271.png)

- setex(set with expire)键秒值/setnx(set if not exist)

- mset/mget/msetnx

- getset(先get再set)

### List

单值多value，可重复

#### 常用

| 命令                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| BLPOP key1 [key2 ] timeout            | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOP key1 [key2 ] timeout            | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOPLPUSH source destination timeout | 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| LINDEX key index                      | 通过索引获取列表中的元素                                     |
| LINSERT key BEFORE/AFTER pivot value  | 在列表的元素前或者后插入元素                                 |
| LLEN key                              | 获取列表长度                                                 |
| LPOP key                              | 移出并获取列表的第一个元素                                   |
| LPUSH key value1 [value2]             | 将一个或多个值插入到列表头部                                 |
| LPUSHX key value                      | 将一个值插入到已存在的列表头部                               |
| LRANGE key start stop                 | 获取列表指定范围内的元素                                     |
| LREM key count value                  | 移除列表元素                                                 |
| LSET key index value                  | 通过索引设置列表元素的值                                     |
| LTRIM key start stop                  | 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。 |
| RPOP key                              | 移除列表的最后一个元素，返回值为移除的元素。                 |
| RPOPLPUSH source destination          | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回     |
| RPUSH key value1 [value2]             | 在列表中添加一个或多个值                                     |
| RPUSHX key value                      | 为已存在的列表添加值                                         |

#### 案例

- lpush/rpush/lrange
- lpop/rpop
- lindex，按照索引下标获得元素(从上到下)
- llen
- lrem key 删N个value
- ltrim key 开始index 结束index，截取指定范围的值后再赋值给key
- rpoplpush 源列表 目的列表
- lset key index value
- linsert key before/after 值1 值2

性能总结：

- 它是一个字符串链表，left、right都可以插入添加；
- 如果键不存在，创建新的链表；
- 如果键已存在，新增内容；
- 如果值全移除，对应的键也就消失了。
- 链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

### Set

单值多value，不可重复

#### 常用

| 命令                                           | 描述                                                |
| ---------------------------------------------- | --------------------------------------------------- |
| SADD key member1 [member2]                     | 向集合添加一个或多个成员                            |
| SCARD key                                      | 获取集合的成员数                                    |
| SDIFF key1 [key2]                              | 返回给定所有集合的差集                              |
| SDIFFSTORE destination key1 [key2]             | 返回给定所有集合的差集并存储在 destination 中       |
| SINTER key1 [key2]                             | 返回给定所有集合的交集                              |
| SINTERSTORE destination key1 [key2]            | 返回给定所有集合的交集并存储在 destination 中       |
| SISMEMBER key member                           | 判断 member 元素是否是集合 key 的成员               |
| SMEMBERS key                                   | 返回集合中的所有成员                                |
| SMOVE source destination member                | 将 member 元素从 source 集合移动到 destination 集合 |
| SPOP key                                       | 移除并返回集合中的一个随机元素                      |
| SRANDMEMBER key [count]                        | 返回集合中一个或多个随机数                          |
| SREM key member1 [member2]                     | 移除集合中一个或多个成员                            |
| SUNION key1 [key2]                             | 返回所有给定集合的并集                              |
| SUNIONSTORE destination key1 [key2]            | 所有给定集合的并集存储在 destination 集合中         |
| SSCAN key cursor [MATCH pattern] [COUNT count] | 迭代集合中的元素                                    |

#### 案例

- sadd/smembers/sismember

	可以sadd重复的，但是在set里只存在一个

	![image-20210219232932994](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210219232932994.png)

- scard，获取集合里面的元素个数

- srem key value 删除集合中元素

- srandmember key 某个整数(随机出几个数)

- spop key 随机出栈

- smove key1 key2 在key1里某个值 作用是将key1里的某个值赋给key2

- 数学集合类   

	- 差集：sdiff key1 key2 在key1里但是不在key2里
	- 交集：sinter key1 key2
	- 并集：sunion key1 key2 