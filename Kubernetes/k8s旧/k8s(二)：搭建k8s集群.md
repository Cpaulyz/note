# k8s(二)：搭建k8s集群

## 1 平台规划

### 1.1 单master集群

单个master节点，管理多个node节点

缺点：master节点挂了就无法管理了

![image-20210127135926744](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210127135926744.png)

### 1.2 多matser集群

多个master节点，管理多个node节点，同时中间多了一个负载均衡的过程

![image-20210127140041681](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210127140041681.png)

## 2 服务器硬件配置要求

* **测试环境**	

	master：2核  4G  20G

	node：   4核  8G  40G

* **生产环境**

	master：8核  16G  100G

	node：   16核  64G  200G

目前生产部署Kubernetes集群主要有两种方式

* **kubeadm**

	kubeadm是一个K8S部署工具，提供kubeadm init 和 kubeadm join，用于快速部署Kubernetes集群

	官网地址：[点我传送](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

* **二进制包**

	从github下载发行版的二进制包，手动部署每个组件，组成Kubernetes集群。

	Kubeadm降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署Kubernetes集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。

## 3 搭建k8s集群部署方式

固定IP：https://blog.csdn.net/readiay/article/details/50866709

部署手册：https://gitee.com/moxi159753/LearningNotes/tree/master/K8S/3_%E4%BD%BF%E7%94%A8kubeadm%E6%96%B9%E5%BC%8F%E6%90%AD%E5%BB%BAK8S%E9%9B%86%E7%BE%A4

