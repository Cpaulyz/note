# basic

## 三种IO

* **IO**

	一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）**

	我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。

	当应用程序发起 I/O 调用后，会经历两个步骤：

	1. 内核等待 I/O 设备准备好数据
	2. 内核将数据从内核空间拷贝到用户空间。

* **BIO（blocking IO）**

	同步阻塞IO

	应用程序发起 read 调用后，会一直阻塞，直到在内核把数据拷贝到用户空间

	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210312165421869.png" alt="image-20210312165421869" style="zoom: 50%;" />

* **NIO(Non-blocking/New I/O)**

  同步非阻塞IO

  **IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗**

  通过选择器监听通道，非阻塞，处理完以后返回

  1. 轮询

  	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210312170644280.png" alt="image-20210312170644280" style="zoom:50%;" />

  2. IO多路复用

  	准备好了再通知

  	<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210312170701911.png" alt="image-20210312170701911" style="zoom:50%;" />

* **AIO**

	异步IO

	通过回调的方式

[漫话：如何给女朋友解释什么是Linux的五种IO模型？](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41&scene=21#wechat_redirect)

[京东数科二面：常见的 IO 模型有哪些？Java 中的 BIO、NIO、AIO 有啥区别？](https://www.cnblogs.com/javaguide/p/io.html)