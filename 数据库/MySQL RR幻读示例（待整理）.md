# MySQL RR幻读示例（待整理）

https://github.com/Yhzhtk/note/issues/42#issuecomment-424079787

https://tech.meituan.com/2014/08/20/innodb-lock.html

之前看面经的时候，都说MySQL下，MVCC在RR隔离级别下可以防止幻读，但是严格来讲并不是这样的。下面是一个反例

执行顺序

![image-20210422200927736](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422200927736.png)

如果要真正实现防止幻读，其实不能用快照读，而要用当前读。

还是按照刚刚的顺序，我们改为当前读

![image-20210422201409948](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422201409948.png)

可以看到，右边直接阻塞了

左边update，右边会直接报错

![image-20210422201503840](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422201503840.png)