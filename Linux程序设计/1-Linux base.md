# 1-Linux base

## what is Linux

https://www.linux.org/

开源、免费的Unix-type操作系统，由GNU分发，特点有

* 开源
* 受欢迎
* 多平台支持

First Version: By Linus (Linus's Unix)

GNU:提倡开发者开发开源的软件，包括编译器、浏览器、库。。。 （GPL协议）

## Install Linux

主要将分区和BootLoader

### 分区理论

![image-20210301154330320](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210301154330320.png)

磁盘组织方式

* MBR: master boot record

	限制：最大只能4T

	![image-20210301154315617](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210301154315617.png)

* GPT: guid partition table

	![image-20210301154248379](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210301154248379.png)