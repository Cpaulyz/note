# Kubernetes(一): 基本资源概念及kubectl

[TOC]

## 1 K8S架构

> 总结来看，**K8S 的 Master Node 具备：请求入口管理（API Server），Worker Node 调度（Scheduler），监控和自动调节（Controller Manager），以及存储功能（etcd）；而 K8S 的 Worker Node 具备：状态和监控收集（Kubelet），网络和负载均衡（Kube-Proxy）、保障容器化运行环境（Container Runtime）、以及定制化功能（Add-Ons）。**

![image-20210812201228020](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210812201228020.png)

* 主节点一般被称为**Master Node 或者 Head Node**（本文采用 Master Node 称呼方式）
* 从节点则被称为**Worker Node 或者 Node**（本文采用 Worker Node 称呼方式）

### Master Node

- **API Server**。**K8S 的请求入口服务**。API Server 负责接收 K8S 所有请求（来自 UI 界面或者 CLI 命令行工具），然后，API Server 根据用户的具体请求，去通知其他组件干活。
- **Scheduler**。**K8S 所有 Worker Node 的调度器**。当用户要部署服务时，Scheduler 会选择最合适的 Worker Node（服务器）来部署。
- **Controller Manager**。**K8S 所有 Worker Node 的监控器**。Controller Manager 有很多具体的 Controller，在文章**[Components of Kubernetes Architecture](https://link.zhihu.com/?target=https%3A//medium.com/%40kumargaurav1247/components-of-kubernetes-architecture-6feea4d5c712)**中提到的有 Node Controller、Service Controller、Volume Controller 等。Controller 负责监控和调整在 Worker Node 上部署的服务的状态，比如用户要求 A 服务部署 2 个副本，那么当其中一个服务挂了的时候，Controller 会马上调整，让 Scheduler 再选择一个 Worker Node 重新部署服务。
- **etcd**。**K8S 的存储服务**。etcd 存储了 K8S 的关键配置和用户配置，K8S 中仅 API Server 才具备读写权限，其他组件必须通过 API Server 的接口才能读写数据（见**[Kubernetes Works Like an Operating System](https://link.zhihu.com/?target=https%3A//thenewstack.io/how-does-kubernetes-work/)**）。

### Worker Node

- **Kubelet**。**Worker Node 的监视器，以及与 Master Node 的通讯器**。Kubelet 是 Master Node 安插在 Worker Node 上的“眼线”，它会定期向 Worker Node 汇报自己 Node 上运行的服务的状态，并接受来自 Master Node 的指示采取调整措施。
- **Kube-Proxy**。**K8S 的网络代理**。私以为称呼为 Network-Proxy 可能更适合？Kube-Proxy 负责 Node 在 K8S 的网络通讯、以及对外部网络流量的负载均衡。
- **Container Runtime**。**Worker Node 的运行环境**。即安装了容器化所需的软件环境确保容器化程序能够跑起来，比如 Docker Engine。大白话就是帮忙装好了 Docker 运行环境。
- **Logging Layer**。**K8S 的监控状态收集器**。私以为称呼为 Monitor 可能更合适？Logging Layer 负责采集 Node 上所有服务的 CPU、内存、磁盘、网络等监控项信息。
- **Add-Ons**。**K8S 管理运维 Worker Node 的插件组件**。有些文章认为 Worker Node 只有三大组件，不包含 Add-On，但笔者认为 K8S 系统提供了 Add-On 机制，让用户可以扩展更多定制化功能，是很不错的亮点。

## 2 K8S概念

### 2.1 Pod

> **Pod**是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

* Pod是K8S服务的一个闭包
* Pod可以理解为一组**共享网络、存储和计算资源**的服务的集合
  * e.g. 同一个 Pod 之间的 Container 可以通过 localhost 互相访问，并且可以挂载 Pod 内所有的数据卷；但是不同的 Pod 之间的 Container 不能用 localhost 访问，也不能挂载其他 Pod 的数据卷
  * ![image-20210812202103341](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210812202103341.png)

```yaml
# 都是v1
apiVersion: v1 
# 对象类型
kind: Pod 
# Pod自身的元数据
metadata: 
  name: memory-demo # pod名称
  namespace: mem-example # 命名空间
# Pod内部资源的详细信息
spec: 
	# Pod内容器信息
  containers:
  - name: memory-demo-ctr # 名称
    image: polinux/stress # 镜像地址
    resources: # 容器需要的 CPU、内存、GPU 等资源
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"] # 容器入口命令
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"] # 容器入口参数
    volumeMounts: # 容器要挂载的 Pod 数据卷等
    - name: redis-storage
      mountPath: /data/redis
  # 上述均为启动容器必要和必须的信息 
  volumes: # Pod内数据卷信息，详见后文
  - name: redis-storage
    emptyDir: {}
```

### 2.2. Volume

Container 中的文件在磁盘上是临时存放的，这给 Container 中运行的较重要的应用 程序带来一些问题。问题之一是当容器崩溃时文件丢失。kubelet 会重新启动容器， 但容器会以干净的状态重启。 第二个问题会在同一 `Pod` 中运行多个容器并共享文件时出现。 Kubernetes [卷（Volume）](https://kubernetes.io/zh/docs/concepts/storage/volumes/) 这一抽象概念能够解决这两个问题。

* 临时卷类型的生命周期与 Pod 相同，当 Pod 不再存在时，Kubernetes 也会销毁临时卷
* 持久卷可以比 Pod 的存活期长，当 Pod 不再存在时候，Kubernetes 不会销毁持久卷
* 对于给定 Pod 中任何类型的卷，在容器重启期间数据都不会丢失。

#### vs. volumeMounts

volume 是 K8S 的对象，对应一个实体的数据卷；

而 volumeMounts 只是 container 的挂载点，对应 container 的其中一个参数。

但是，volumeMounts 依赖于 volume，只有当 Pod 内有 volume 资源的时候，该 Pod 内部的 container 才可能有 volumeMounts。

### 2.3 Container

> Pod 下的一个容器，一个Pod可以有多个Container

- 标准容器 Application Container。
- 初始化容器 Init Container。
- 边车容器 Sidecar Container。
- 临时容器 Ephemeral Container。

一般来说，我们部署的大多是**标准容器（ Application Container）**。

### 2.4 Deployment 和 ReplicaSets

> 一个 *Deployment* 控制器为 **[Pods](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/pods/pod-overview/)** 和 **[ReplicaSets](https://link.zhihu.com/?target=https%3A//kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)** 提供声明式的更新能力。
> 你负责描述 Deployment 中的 *目标状态*，而 Deployment 控制器以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment，并通过新的 Deployment 收养其资源。

翻译一下：**Deployment 的作用是管理和控制 Pod 和 ReplicaSet，管控它们运行在用户期望的状态中**。哎，打个形象的比喻，**Deployment 就是包工头**，主要负责监督底下的工人 Pod 干活，确保每时每刻有用户要求数量的 Pod 在工作。如果一旦发现某个工人 Pod 不行了，就赶紧新拉一个 Pod 过来替换它。

新的问题又来了：那什么是 ReplicaSets 呢？

> ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

再来翻译下：ReplicaSet 的作用就是管理和控制 Pod，管控他们好好干活。但是，ReplicaSet 受控于 Deployment。形象来说，**ReplicaSet 就是总包工头手下的小包工头**。

![image-20210812203927399](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210812203927399.png)

从 K8S 使用者角度来看，用户会直接操作 Deployment 部署服务，而当 Deployment 被部署的时候，K8S 会自动生成要求的 ReplicaSet 和 Pod。在**[K8S 官方文档](https://link.zhihu.com/?target=https%3A//www.kubernetes.org.cn/replicasets)**中也指出用户只需要关心 Deployment 而不操心 ReplicaSet：

> This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.
> 这实际上意味着您可能永远不需要操作 ReplicaSet 对象：直接使用 Deployments 并在规范部分定义应用程序。

补充说明：在 K8S 中还有一个对象 --- **ReplicationController（简称 RC）**，**[官方文档](https://link.zhihu.com/?target=https%3A//kubernetes.io/zh/docs/concepts/workloads/controllers/replicationcontroller/)**对它的定义是：

> *ReplicationController* 确保在任何时候都有特定数量的 Pod 副本处于运行状态。 换句话说，ReplicationController 确保一个 Pod 或一组同类的 Pod 总是可用的。

怎么样，和 ReplicaSet 是不是很相近？在**[Deployments, ReplicaSets, and pods](https://link.zhihu.com/?target=https%3A//www.ibm.com/cloud/architecture/content/course/kubernetes-101/deployments-replica-sets-and-pods/)**教程中说“ReplicationController 是 ReplicaSet 的前身”，官方也推荐用 Deployment 取代 ReplicationController 来部署服务。

### 2.5 Service 和 Ingress

前文介绍的 Deployment、ReplicationController 和 ReplicaSet 主要管控 Pod **程序服务**；

Service 和 Ingress 则负责管控 Pod **网络服务**。

#### Service

> 将运行在一组 **[Pods](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/pods/pod-overview/)** 上的应用程序公开为网络服务的抽象方法。
> 使用 Kubernetes，您无需修改应用程序即可使用不熟悉的服务发现机制。 Kubernetes 为 Pods 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。

Service 不是”服务“，更像是一个网关层，是Pod的流量入口，可以做负载均衡等工作。

* 为什么需要 Service？
  * Kubernetes **[Pod](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/pods/pod-overview/)** 是有生命周期的。 它们可以被创建，而且销毁之后不会再启动。 如果您使用 **[Deployment](https://link.zhihu.com/?target=https%3A//kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)** 来运行您的应用程序，则它可以动态创建和销毁 Pod。
  * 每个 Pod 都有自己的 IP 地址，但是在 Deployment 中，在同一时刻运行的 Pod 集合可能与稍后运行该应用程序的 Pod 集合不同。
  * 这导致了一个问题： 如果一组 Pod（称为“后端”）为群集内的其他 Pod（称为“前端”）提供功能， 那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用工作量的后端部分？

* **Service 是 K8S 服务的核心，屏蔽了服务细节，统一对外暴露服务接口，真正做到了“微服务”**
  * 举个例子，我们的一个服务 A，部署了 3 个备份，也就是 3 个 Pod；对于用户来说，只需要关注一个 Service 的入口就可以，而不需要操心究竟应该请求哪一个 Pod。优势非常明显：
    * 一方面外部用户不需要感知因为 Pod 上服务的意外崩溃、K8S 重新拉起 Pod 而造成的 IP 变更，外部用户也不需要感知因升级、变更服务带来的 Pod 替换而造成的 IP 变化
    * 另一方面，Service 还可以做流量负载均衡。

#### Ingress

> Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。
> Ingress 可以提供负载均衡、SSL 终结和基于名称的虚拟托管。

* Service 负责集群内的网络拓扑，Ingress负责使集群外部访问集群内部

* 也就是说，Ingress 是整个 K8S 集群的接入层，复杂集群内外通讯

![image-20210813102614276](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813102614276.png)

#### 实践

参考：k8s外网如何访问业务应用https://www.jianshu.com/p/50b930fa7ca3

创建两个ngixn pod，开放端口80

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % vim nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx  
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx  # 就是说哪些Pod被Service池化是根据Label标签来的，此行nginx字样，后面我们创建Service会用到
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80


kubectl create -f nginx.yaml 
deployment.apps/nginx-deployment created
```

![image-20210813103843216](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813103843216.png)

创建 Service 池化刚刚两个 Pods

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % vim nginx-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080  # 这个资源(svc)开放的端口
    targetPort: 80
# selector选择之前Label标签为nginx的Pod作为Service池化的对象，
# 最后说的是把Service的8080端口映射到Pod的80端口。

chenyanze@chenyanzedeMacBook-Pro k8s % kubectl apply -f nginx-svc.yaml                         
service/nginx-svc created
```

![image-20210813104021231](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813104021231.png)

可以通过 describe 命令来查看具体的 service 状态

这里有几个Port需要区分

* port是service的端口 
* targetport是pod（也就是容器）的端口 
* nodeport是容器所在node节点的端口（实质上也是通过nodeport类型的service暴露给集群节点，但port没有service类型

![image-20210813104211891](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813104211891.png)

### 2.6 Namespace

> Kubernetes 支持多个虚拟集群，它们底层依赖于同一个物理集群。 这些虚拟集群被称为名字空间。

Namespace 是用于服务整个K8s集群的，可以将一个K8s集群划分为若干个资源不可共享的虚拟集群

* 比如我有 2 个业务 A 和 B，那么我可以创建 ns-a 和 ns-b 分别部署业务 A 和 B 的服务，如在 ns-a 中部署了一个 deployment，名字是 hello，返回用户的是“hello a”；在 ns-b 中也部署了一个 deployment，名字恰巧也是 hello，返回用户的是“hello b”（要知道，在同一个 namespace 下 deployment 不能同名；但是不同 namespace 之间没有影响）
* 前文提到的所有对象，都是在 namespace 下的；当然，也有一些对象是不隶属于 namespace 的，而是在 K8S 集群内全局可见的
  * 可以通过以下命令来列出支持/不支持namespace的资源

```shell
# 位于名字空间中的资源
kubectl api-resources --namespaced=true

# 不在名字空间中的资源
kubectl api-resources --namespaced=false
```

## 3 kubectl 操作

### 3.1 更新/修改资源

* **方法一：修改 yaml 文件后通过 kubectl 更新**。我们看到，创建一个 Pod 或者 Deployment 的命令是`kubectl apply -f ${YAML}`
* **方法二：通过 kubectl 直接编辑 K8S 上的服务**。命令为`kubectl edit ${RESOURCE} ${NAME}`，比如修改刚刚的 Pod 的命令为`kubectl edit pod memory-demo`，然后直接编辑自己要修改的内容即可

### 3.2 删除资源

在 K8S 上删除服务的操作非常简单，命令为`kubectl delete ${RESOURCE} ${NAME}`。比如删除一个 Pod 是：`kubectl delete pod memory-demo`，再比如删除一个 Deployment 的命令是：`kubectl delete deployment ${DEPLOYMENT_NAME}`

需要注意的是，通过 Deployment 部署的服务，那么仅仅删除 Pod 是不行的，正确的删除方式应该是：先删除 Deployment，再删除 Pod

有时候会发现一个 Pod 总也删除不了，这个时候很有可能要实施强制删除措施，命令为`kubectl delete pod --force --grace-period=0 ${POD_NAME}`。

### 4 How to debug

Kubernetes Deployment故障排除图解指南https://tonybai.com/2019/12/08/k8s-deployment-troubleshooting/

- 查看 Pod 内部某个 container 打印的日志：`kubectl log ${POD_NAME} -c ${CONTAINER_NAME}`。
- 进入 Pod 内部某个 container：`kubectl exec -it [options] ${POD_NAME} -c ${CONTAINER_NAME} [args]`，嗯，这个命令的作用是通过 kubectl 执行了`docker exec xxx`进入到容器实例内部。之后，就是用户检查自己服务的日志来定位问题。

## reference

https://zhuanlan.zhihu.com/p/339008746
