# docker

容器和虚拟机的区别？

容器更轻量级，启动更快

## 判断

容器镜像是分层的，只有最上面一层可读写，其他可读  √

虚拟机是操作系统级别的资源隔离，容器本质上是进程级的资源隔离

## 参数

-P 挑一个端口映射

docker port 查看端口映射

--name 起名字

-d 后台运行

-e 传环境变量



## 管理命令

docker network

* 如何创建、管理网络
* 如果一个容器在两个网络里，分别有一个IP地址

docker container

docker image



在容器里看IP地址

* `cat /etc/hosts`