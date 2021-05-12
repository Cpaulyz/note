# SQL

### B树结构

* 结构

  1. 非叶子节点只存储键值信息，所有记录节点都是按键值的大小顺序存放在同一层的叶子节点上
  2. 各叶子节点通过指针进行连接

* 能做什么

	* 全键值查询，如`where x=123`
	* 键范围查询，如`where 45<x<123`
	* 键前缀查询，如`where x like "J%"`

* 不能做什么

	后缀查询
	
	对于复合索引，如果违反最左匹配原则，则不能用

> ![image-20210427183816352](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427183816352.png)
>
> 1. 结构：所有记录节点都是按键值的大小顺序存放在同一层的叶子节点上；各叶子节点通过指针进行连接
>
> 	使用方式：查找时会从根节点读取磁盘块，自顶向下根据B树结构进行查找，对于范围查询，可以通过叶子节点的指针减少IO
>
> 2. 前缀查询；键范围查询；全键值查询
>
> 3. 避免死锁
>
> 	Oracle中，对从表执行DML，主键列会加共享锁，可能导致其他事务出现阻塞，进而发生死锁
>
> 	提高性能
>
> 	以MySQL为例，InnoDB提供了四种外键关联策略
>
> 	* RESTRICT（默认）：父表在删除和更新记录的时候，要在子表中检查是否有有关该父表要更新和删除的记录，如果有，则不允许删除和更改
> 	* CASCADE：级联删除或更改
> 	* NO ACTION：什么也不做
> 	* SET NULL：置空，当父表更新、删除的时候，字表会把外键字段变为null，所以这个时候设计表的时候该字段要允许为null，否则会出错
>
> 	可以看出，在外键字段上，如果主表发生了变化，通常会对从表进行修改，这就需要在外键字段上进行查询，如果没有加上索引，需要对从表进行全表遍历
>
> 4. 主表不会改变时
>
> 5. 结构：索引组织表的数据按主键排序手段被存储在B-树索引中，除了存储主键列值外还
>
> 	存储非键列的值。普通索引只存储索引列，而索引组织表则存储表的所有列的值。
> 	
> 	使用：完全由主键组成的表；代码查找表，只会通过一个主键来访问一个表；想保证数据存储在某个位置上，或者希望数据以某种特定的顺序物理存储

### MyISAM vs InnoDB

* MyISAM使用前缀压缩以减少索引，而InnoDB不会压缩索引，（有啥差别？） 

	https://www.cnblogs.com/xiaoboluo768/p/5166939.html

* MyISAM索引按照行存储的物理位置引用被索引的行，但是InnDB按照主键值引用行，（有啥差别?

	https://blog.csdn.net/qq_35642036/article/details/82820178

### 如何让索引发挥作用

* 复合索引

	本质上索引是按照排名第一的字段进行的索引，最左匹配

* 回表查询

	使用聚集索引，或者查询字段被索引覆盖时候，可以只使用索引，不使用表

### 索引与外键

系统地对表的外键加上索引的做法非常普遍

* 为什么

  * 避免死锁

    Oracle中，对从表执行DML，主键列会加共享锁，可能导致其他事务出现阻塞，进而发生死锁

  * 提高性能

  	以MySQL为例，InnoDB提供了四种外键关联策略

  	* RESTRICT（默认）：父表在删除和更新记录的时候，要在子表中检查是否有有关该父表要更新和删除的记录，如果有，则不允许删除和更改
  	* CASCADE：级联删除或更改
  	* NO ACTION：什么也不做
  	* SET NULL：置空，当父表更新、删除的时候，字表会把外键字段变为null，所以这个时候设计表的时候该字段要允许为null，否则会出错

  	可以看出，在外键字段上，如果主表发生了变化，通常会对从表进行修改，这就需要在外键字段上进行查询，如果没有加上索引，需要对从表进行全表遍历

* 有例外吗？

	主表不会改变时

> ![image-20210427183116640](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427183116640.png)
>
> 1. 其索引使用bit数组（或称bitmap、bit set、bit string、bit vector）进行存储与计算操作。主要针对大量相同值的列而创建，一个索引行中存储键值和起止Rowid,以及这些键值的位置编码,位置编码中的每一位表示键值对应的数据行的有无
>
> 	用途：相异基数低、大量临时查询的聚合
>
> 2. 对F(x)的值构建索引，在通过索引读取x所指向的记录行。
>
> 	用途：如不区分大小写的查询
>
> 3. 反向键索引是特殊的B树索引，适用于在表中严格排序的列上创建反向键索引，查询时只要像常规方式一样查询数据，不需要关心反向键处理，因为oracle会自动完成该处理
>
> 	用途：减少插入高并发下，主键索引创建时的资源竞争
>
> 4. B+树、哈希、索引组织表、二叉搜索树...

### 索引组织表*

https://www.cnblogs.com/nieliu/archive/2012/05/04/2482223.html

索引组织表(index organized table, IOT)就是存储在一个索引结构中的表。存储在堆中的表是无组织的(也就是说，只要有可用的空间，数据可以放在任何地方)，IOT中的数据则按主键存储和排序。对你的应用来说，IOT表和一个“常规”表并无二致。

索引组织表的数据按主键排序手段被存储在B-树索引中，除了存储主键列值外还存储非键列的值。普通索引只存储索引列，而索引组织表则存储表的所有列的值。

索引组织表一般适应于静态表，且查询多以主键列。当表的大部分列当作主键列时，且表相对静态，比较适合创建索引组织表！（8i以上）

既然它属于表，那么它当然也有建立索引的需求。由于它的索引的结构，比如说由于索引叶节点的分裂，行所在块可能会发生改变，因而建立在IOT上的索引和一般的索引的最大区别是它存的是IOT的行的逻辑地址，也就是UROWID，oracle用这个逻辑rowid来猜这个行所在的块，如果猜到了，那么这个urowid是正确的，否则它从这个地址向下遍历来找这条记录。

IOT表的rowid是逻辑上的，因为IOT表中的行的位置是在不断变化的(例如插入新的行，有可能带来其它行的位置移动)

  IOT有什么意义呢？使用堆组织表时，我们必须为表和表主键上的索引分别留出空间。而IOT不存在主键的空间开销，因为索引就是数据，数据就是索引，二者已经合二为一。但是，IOT带来的好处并不止于节约了磁盘空间的占用，更重要的是大幅度降低了I/O,减少了访问缓冲区缓存(尽管从缓冲区缓存获取数据比从硬盘读要快得多，但缓冲区缓存并不免费，而且也绝对不是廉价的。每个缓冲区缓存获取都需要缓冲区缓存的多个闩，而闩是串行化设备，会限制应用的扩展能力)

   IOT适用的场合有：
 1、完全由主键组成的表。这样的表如果采用堆组织表，则表本身完全是多余的开销，因为所有的数据全部同样也保存在索引里，此时，堆表是没用的。
 2、代码查找表。如果你只会通过一个主键来访问一个表，这个表就非常适合实现为IOT.
 3、如果你想保证数据存储在某个位置上，或者希望数据以某种特定的顺序物理存储，IOT就是一种合适的结构。

  IOT提供如下的好处：
 ·提高缓冲区缓存效率，因为给定查询在缓存中需要的块更少。
 ·减少缓冲区缓存访问，这会改善可扩缩性。
 ·获取数据的工作总量更少，因为获取数据更快。	 
 ·每个查询完成的物理I/O更少，因为对于任何给定的查询，需要的块更少，而且对地址记录的一个物理 I/O 很可能可以获取所有地址（而不只是其中一个地址，但堆表实现就只是获取一个地址）

  如果经常在一个主键或惟一键上使用BETWEEN 查询也是如此，因为相近的记录存在一起，查询时引入的逻辑IO和物理IO都会更少。

### 系统生成键

* 系统生产序列号，远好于 
	* 寻找当前最大值并加1 
	* 用一个专用表保存”下一个值“且加锁更新
* 但如果插入并发性过高，在主键索引的创建操作上会发生十分严重的资源竞争
* 解决方案
  * 反向键索引或叫逆向索引（reverse index）
  	* 反向键索引是特殊的B数索引 
  	* 适用于在表中严格排序的列上创建反向键索引 
  	* 查询时只要像常规方式一样查询数据，不需要关心反向键处理，因为oracle会自动完成该处理
  * 哈希索引（hash indexing

### SQL执行顺序

![image-20210422174146262](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422174146262.png)

* 最复杂：解析（最耗资源，但不是最重要的

	绑定变量：https://blog.csdn.net/maray/article/details/7663598

* 最重要：执行

### 绑定变量

提到绑定变量，首先肯定想到硬解析和软解析。绑定变量时解决硬解析的利器。

* 硬解析：就是一条没有执行过的sql。数据库首先对他进行语法分析和解析，过后，根据分析的信息生成最好的执行计划，然后执行。

* 软解析：就是已经存在了一样的sql语句了

绑定变量实质就是变量。类似于我们是用过的替代变量（占位符）。就是在sql语句中使用变量，通过改变变量的值来得到不同的结果。

sql语句是分为动态部分和静态部分的。而动态部分在一般的情况下，对执行计划的影响是微乎其微的。所以同一个sql语句有不同动态部分生成的执行计划是相同的。

* 优点：

	使用动态绑定，可以减少sql的解析，从而减少了数据库引擎在sql解析上资源的消耗。提高了执行效率和可靠性。减少对数据库的访问实际上就是减少了数据库的工作量

* 缺点：

	可能长时间使用动态sql，由于参数的不同。可能sql的执行效率不同。

> ![image-20210427182726815](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427182726815.png)
>
> 1. 语法语义检查、解析、执行计划、执行引擎、存储引擎、数据库
>
> 2. 硬解析：就是一条没有执行过的sql。数据库首先对他进行语法分析和解析，过后，根据分析的信息生成最好的执行计划，然后执行。
>
> 	软解析：因为相同文本的SQL语句存在于library cache中，所以本次SQL语句的解析就可以去掉硬解析中的一个或多个步骤。从而节省大量的资源的耗费
>
> 3. 绑定变量
>
> 4. ```java
> 	pstmt = con.prepareStatement("UPDATE employees SET salay = ? WHERE id = ?"); pstmt.setBigDecimal(1, 150.00); 
> 	pstmt.setInt(2, id); 
> 	pstmt.executeQuery(); 
> 	```

### SQL优化

* 影响因素

	* 结果集的大小

		取决于表的大小和过滤条件的细节

	* 表的数量

		对于优化器，随着表数量的增加，复杂度呈指数增长

		编写太多复杂查询时，多种方式的连接选择失误率高

* 过滤条件，满足越少越好，-》 用exist in 来暗示

	基本原则：

	* 外层条件好，用exists
	* 外层条件不好，用in
	* 不知道的情况下，用in

* 降低表连接数量-》改写SQL、 打破范式

> 简述查询优化器的工作原理和局限性

### 隔离级别（必考简答）

PPT

### 代码

![image-20210427184418506](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427184418506.png)

1. ```sql
	select s.sname
	from Sailors s
	where s.age>35 and s.rating>5 and s.sid not exists(
		select r.sid
		from Boats b, Reaserves r
		where b.bid=r.bid and b.color='Red' 	
		and r.sid=s.sid and DATEDIFF(current_date(),r.day)<30 
	);
	```

2. ```sql
	select s.sname
	from Sailors s
	where s.age>35 and s.rating>5 and s.sid in(
		select r.sid
		from Boats b, Reaserves r
		where b.bid=r.bid and b.color='Red' and DATEDIFF(current_date(),r.day)<30 
	)and s.sid in(
		select r.sid
		from Boats b, Reaserves r
		where b.bid=r.bid and b.color='Green' and DATEDIFF(current_date(),r.day)<30 
	);
	```

3. ```sql
	select s.sname from Sailors s where not exists(
	    select * from Boats b where not exists(
	        select * from Reaserves r 
	        where r.bid=b.bid and r.sid=s.sid
	    )
	);
	```

当s.age>35 and s.rating>5过滤条件足够好时，使用exists效率高；否则使用in

![image-20210427185526093](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427185526093.png)

1. * 邻接模型

		每一条记录用一个额外的列来标识父节点

	* 物化路径模型

		使用层次式的路径明确地标识出来层次结构，路径一般保存为字符串格式，如1.1，1.1.1，1.1.2，...

	* 嵌套集合模型

		使用左右界来识别层次关系，子节点的左右界包含于父节点

2. 例如，属性为description和commander的列

	```
		a
	  /   \
	 b	   c
	```

	* 邻接模型：

		```
		id parent_id description commander
		1	0			a		a
		2	1			b		b
		3	1			c		c
		```

	* 物化路径

		```
		path description commander
		1		a		a
		1.1		b		b
		1.2		c		c
		```

	* 嵌套集合模型

		```
		description commander left right
			a		a		1	100
			b		b		2	49
			c		c		50	99
		```

3. * 邻接模型

		```sql
		select description,commander
		from table1
		connect by parent_id = prior id
		start with commander='a';
		```

	* 物化路径

		```sql
		select t2.description,t2.commander
		from table2 t1,table2 t2
		where t1.commander='a' and
		t2.path = t1.path || "%";
		```

	* 嵌套集合

		```sql
		select t2.description,t2.commander
		from table3 t1,table3 t2
		where t1.commander='a' and
		t2.left>t1.left and t2.right<t2.right
		```

4.  * 邻接模型

		```sql
		select description,commander
		from table1
		connect by id = prior parent_id
		start with commander='b';
		```

	* 物化路径

	  ```sql
	  select t1.description,t1.commander
	  from table2 t1,table2 t2
	  where t1.commander='b' and
	  t2.path = t1.path || "%";
	  ```

	* 嵌套集合

	  ```sql
	  select t1.description,t1.commander
	  from table3 t1,table3 t2
	  where t1.commander='b' and
	  t2.left>t1.left and t2.right<t2.right
	  ```
	
5. * 邻接模型的两种查询效率差不多，因为都是使用了数据库的递归查询函数，connect by函数的实现不是基于关系，而是基于过程，是提取所有相关记录再处理
   * 物化路径自底向上的性能降低很多，因为自顶向下只从一个节点出发，而自底向上需要从先通过字符串处理找到多个符合的子代节点，再分别求其祖先节点
   * 虽然邻接模型要做一次迭代，效率影响较大，却又因为是数值比较，其速度反而是最快的；物化路径模型虽然比较简单，但是其需要拆分字符串并重新寻找深度等，字符串运算会导致压力比较大

![image-20210427183041510](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210427183041510.png)