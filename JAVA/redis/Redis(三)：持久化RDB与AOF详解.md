# Redis(三)：持久化RDB与AOF详解

[TOC]



## 1 持久化

Redis 提供了不同级别的持久化方式:

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储.
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.
- 如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 你也可以同时开启两种持久化方式, 在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.
- 最重要的事情是了解RDB和AOF持久化方式的不同,让我们以RDB持久化方式开始:

## 2 RDB

> Redis DataBase

### 2.1 是什么

![image-20210222160710331](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160710331.png)

* 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里
* Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到  一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。  整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且==对于数据恢复的完整性不是非常敏感==，那RDB方式要比AOF方式更加的高效。
* RDB的缺点是最后一次持久化后的数据可能丢失。

> Fork
>
> Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程

- rdb 保存的是dump.rdb文件

### 2.2 如何配置

- 相关配置在配置文件的位置 - 在redis.conf搜寻`SNAPSHOTTING`

	```
	################################ SNAPSHOTTING  ################################
	#
	# Save the DB on disk:
	#
	#   save <seconds> <changes>
	#
	#   Will save the DB if both the given number of seconds and the given
	#   number of write operations against the DB occurred.
	#
	#   In the example below the behaviour will be to save:
	#   after 900 sec (15 min) if at least 1 key changed
	#   after 300 sec (5 min) if at least 10 keys changed
	#   after 60 sec if at least 10000 keys changed
	#
	#   Note: you can disable saving completely by commenting out all "save" lines.
	#
	#   It is also possible to remove all the previously configured save
	#   points by adding a save directive with a single empty string argument
	#   like in the following example:
	#
	#   save ""
	```


### 2.3 备份时机

* 根据`redis.conf`中的配置，默认设置是1分钟内修改一万次、或者5分钟内修改10次、或者15分钟内修改1次
* 命令`save`或者是`bgsave   `
	- Save：save时只管保存，其它不管，全部阻塞
	- BGSAVE：Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求。可以通过lastsave 命令获取最后一次成功执行快照的时间
* 执行`flushall`命令，也会产生dump.rdb文件，但里面是空的，无意义

### 2.4 恢复方法

- 将备份文件 (dump.rdb) 移动到 redis 启动目录并启动服务即可
- `CONFIG GET dir`获取启动目录，默认是`./`

### 2.5 优缺点

#### RDB的优点

- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.

- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.

- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.

- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

#### RDB的缺点

- 如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.

- RDB  需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.

### 2.6 实战

为了演示方便，将备份触发条件改为2分钟10条修改

![image-20210222160034410](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160034410.png)

启动redis服务端与客户端

![image-20210222160130010](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160130010.png)

迅速在2分钟内做出一系列更改

![](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160226130.png)

两分钟后，可以看到`dump.rbd`文件产生在启动目录下

![image-20210222160329126](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160329126.png)

在实际开发中，运维会定时备份快照，这里直接cp一个backup作为演示

![image-20210222160417646](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160417646.png)

模拟破坏redis，flushall

![image-20210222160447585](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160447585.png)

此时进行恢复，将backup文件覆盖dump.rdb

![image-20210222160521870](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160521870.png)

> 注意，这时候的dump.rdb由于执行了flushall指令，是一个新生成的空备份，覆盖了旧的快照，旧的快照是dump.rbd.backup，所以需要覆盖

重启服务，发现数据都回来了！

![image-20210222160550591](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222160550591.png)

## 3 AOF

> Append Only File

### 3.1 是什么

![image-20210222163437353](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222163437353.png)

* 以日志的形式来记录**每个写操作**，将Redis执行过的所有写指令记录下来(读操作不记录)， 只许追加文件但不可以改写文件
* redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

### 3.2 如何配置

- 相关配置在配置文件的位置 - 在redis.conf搜寻`### APPEND ONLY MODE ###`，默认关闭
- aof保存的是appendonly.aof文件（在配置文件可修改文件名）

### 3.3 AOF启动/修复/恢复

- 正常恢复   
	- 启动：设置Yes     
		- 修改默认的appendonly no，改为yes
	- 将有数据的aof文件复制一份保存到对应目录(config get dir)
	- 恢复：重启redis然后重新加载
- 异常恢复   
	- 启动：设置Yes     
		- 修改默认的appendonly no，改为yes
	- 备份被写坏的AOF文件
	- 修复：     
		- `redis-check-aof --fix`进行修复
	- 恢复：重启redis然后重新加载
- 每修改同步：appendfsync always 同步持久化 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好
- 每秒同步：appendfsync everysec 异步操作，每秒记录 如果一秒内宕机，有数据丢失
- 不同步：appendfsync no 从不同步

### 3.4 重新Rewrite

因为 AOF 的运作方式是不断地将命令追加到文件的末尾， 所以随着写入命令的不断增加， AOF 文件的体积也会变得越来越大。举个例子，  如果你对一个计数器调用了 100 次 INCR ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100  条记录（entry）。然而在实际上， 只使用一条 SET 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。

为了处理这种情况， Redis 支持一种有趣的特性： 可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。执行  BGREWRITEAOF 命令， Redis 将生成一个新的 AOF 文件， 这个文件包含重建当前数据集所需的最少命令。Redis 2.2  需要自己手动执行 BGREWRITEAOF 命令； Redis 2.4 则可以自动触发 AOF 重写， 具体信息请查看 2.4 的示例配置文件。

### 3.5  优缺点

#### AOF 优点

* 使用AOF 会让你的Redis更加耐久:  你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.

* AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.

* Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF  文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF  文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF  文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。

* AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF  文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子，  如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL  命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

#### AOF 缺点

* 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。

* 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭  fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB  可以提供更有保证的最大延迟时间（latency）。

### 3.6 实战

为方便修改，另外拉一个文件

```
cp /opt/redis-6.0.6/redis.conf /opt/redis-6.0.6/redis_aof.conf
```

开启appendonly

![image-20210222161656048](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222161656048.png)

启动redis，随便写点东西，然后flushall破坏

![image-20210222162720507](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222162720507.png)

可以在当前目录下看到`appendonly.aof`文件

查看后，可以看到每一步的日志，尝试将最后的FLUSHALL删除后重启

![image-20210222162823430](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222162823430.png)

果然，数据又回来了

![image-20210222162854564](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222162854564.png)

尝试在aof文件后面加上一些乱码

![image-20210222162937428](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222162937428.png)

可以发现，启动redis失败，原因是aof无法恢复

![image-20210222163003187](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222163003187.png)

使用`redis-check-aof --fix`修复aof文件

![image-20210222163100573](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222163100573.png)

重启后，发现正常

![image-20210222163113130](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222163113130.png)

## 4 AOF和RDB同时开启

其实非常简单,只需要思考一下，**RDB和AOF 谁的数据更全?**

我们都是知道AOF是基于命令追加,,而RDB是基于快照,根据策略每隔一段时间保存一份数据快照,相比较之下，**AOF更新频率更，数据更加完整,所以如果AOF和RDB同时存在的时候,Redis会优先使用从AOF文件来还原数据库状态**,如果AOF关闭状态时,则从RDB中恢复

![image-20210222162341100](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210222162341100.png)

> 事实上，通过AOF的实战我们也可以看出AOF优先，原因是
>
> * aof损坏时，redis无法启动
> * 手动修改aof文件，删除最后的FLUSHALL后，重启redis，恢复出了数据。如果RDB优先，应该恢复为空

## 参考

[Redis官方网站：持久化](http://www.redis.cn/topics/persistence.html)