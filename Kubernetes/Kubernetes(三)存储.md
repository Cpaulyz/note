# Kubernetes(三): 存储

[TOC]



## 1 Volumne 卷

> Kubernetes的卷是pod的一个组成部分，因此像容器一样在pod的规范中就定义了。它们不是独立的Kubernetes对象，也不能单独创建或删除。pod中的所有容器都可以使用卷，但必须先将它挂载在每个需要访问它的容器中。在每个容器中，都可以在其文件系统的任意位置挂载卷。
>

容器磁盘上的文件的生命周期是短暂的，这就使得在容器中运行重要应用时会出现一些问题。首先，当容器崩溃时，kubelet会重启它，但是容器中的文件将丢失——容器以干净的状态（镜像最初的状态）重新启动。其次，在 Pod 中同时运行多个容器时，这些容器之间通常需要共享文件。Kubernetes 中的 Volume 抽象就很好的解决了这些问题。

k8s支持的卷类型如下图所示

![image-20210813141654905](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813141654905.png)

## 2 emptyDir

EmptyDir 是一个空目录，他的生命周期和所属的 Pod 是完全一致的，它用处是把同一 Pod 内的不同容器之间共享工作过程产生的文件。当 Pod 被分配给节点时，首先创建 emptyDir 卷，并且只要该 Pod 在该节点上运行，该卷就会存在。正如卷的名字所述，它最初是空的。

Pod 中的容器可以读取和写入 emptyDir 卷中的相同文件，尽管该卷可以挂载到每个容器中的相同或不同路径上。当出于任何原因从节点中删除 Pod 时，emptyDir 中的数据将被永久删除。emptyDir 的用法有：

- 暂存空间，例如用于基于磁盘的合并排序
- 用作长时间计算崩溃恢复时的检查点
- Web 服务器容器提供数据时，保存内容管理器容器提取的文件

### 实战

以一个例子进行实战，可以理解为将一个目录挂载到 nginx pod 的 `/usr/share/nginx/html`，挂载到 busybox pod 的`/data/`中，busybox往里面不断写数据，如果能够实现共享，通过nginx访问到的页面应该会不断有新的数据产生。

* 创建pod

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % vim emptyDir_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: prod                           #pod标签
  name: emptydir-fortune
spec:
  containers:
  - name: busybox
    image: busybox:latest
    imagePullPolicy: IfNotPresent
    volumeMounts:
      - name: html
        mountPath: /data/
    command: ["/bin/sh", "-c"]
    args:
      - "while true; do echo $(date) >> /data/index.html; sleep 3; done"
  - image: nginx:alpine
    name: web-server
    volumeMounts:                       #挂载相同的卷至容器/usr/share/nginx/html目录且设置为只读
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html                          #卷名为html的emptyDir卷同时挂载至以上两个容器
    emptyDir: {}

chenyanze@chenyanzedeMacBook-Pro k8s % kubectl get pods     
NAME                                READY   STATUS    RESTARTS   AGE
emptydir-fortune                    2/2     Running   0          81s
```

* 创建service

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % vim emptyDir_svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service           #service名
spec:
  type: NodePort
  selector:
    app: prod                #pod标签,由此定位到pod emptydir-fortune
  ports:
  - protocol: TCP
    port: 8881               #ClusterIP监听的端口
    targetPort: 80           #容器端口
  sessionAffinity: ClientIP  #是否支持Session,同一个客户端的访问请求都转发到同一个后端Pod

chenyanze@chenyanzedeMacBook-Pro k8s % kubectl apply -f emptyDir_svc.yaml 
service/my-service created
```

* 访问nginx，因为我这里用的是minikube，所以直接`minikube service my-service`进行访问

![image-20210813143527334](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813143527334.png)

## 3 hostPath

hostPath允许挂载Node上的文件系统到Pod里面去。如果Pod需要使用Node上的文件，可以使用hostPath。在同一个节点上运行并在其hostPath卷中使用相同路径的pod可以看到相同的文件。

hostPath 卷将主机节点的文件系统中的文件或目录挂载到集群中，hostPath 的用途如下所示：

- 运行需要访问 Docker 内部的容器

- - 使用 /var/lib/docker 的 hostPath

- 在容器中运行 cAdvisor 监控服务

- - 使用 /dev/cgroups 的 hostPath

- 允许 pod 指定给定的 hostPath

- - 是否应该在 pod 运行之前存在，是否应该创建，以及它应该以什么形式存在

除了所需的 path 属性之外，用户还可以为 hostPath 卷指定 type。使用这种卷类型是请注意，因为：

- 由于每个节点上的文件都不同，具有相同配置（例如从 podTemplate 创建的）的 pod 在不同节点上的行为可能会有所不同。
- 当 Kubernetes 按照计划添加资源感知调度时，将无法考虑 hostPath 使用的资源。
- 在底层主机上创建的文件或目录只能由 root 写入。您需要在特权容器中以 root 身份运行进程，或修改主机上的文件权限以便写入 hostPath 卷。

### 实战

* 先在node中创建目录并写点东西，需要注意的是，因为我使用环境是docker起到minikube，所以应该进入到docker中进行创建，而不是直接在本机上创建

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % docker exec -it 9cb7b /bin/bash
root@minikube:/# mkdir /tmp/k8s/ && echo `hostname` > /tmp/k8s/index.html 
root@minikube:/# exit
exit
```

* 创建pod并挂载

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % vim hostPath_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: prod 
  name: hostpath-nginx
spec:
  containers:
  - image: nginx 
    name: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html   #容器挂载点
      name: nginx-volume                 #挂载卷nginx-volume
  volumes:
  - name: nginx-volume                   #卷名
    hostPath:
      path: /tmp/k8s/                        #准备挂载的node上的文件系统
      
chenyanze@chenyanzedeMacBook-Pro k8s % kubectl apply -f hostPath_pod.yaml  
pod/hostpath-nginx created
```

* 验证

```shell
chenyanze@chenyanzedeMacBook-Pro k8s % kubectl exec -it pod/hostpath-nginx /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@hostpath-nginx:/# cat /usr/share/nginx/html/index.html 
minikube
root@hostpath-nginx:/# exit
exit
```

## 4 NFS

NFS是Network File System的缩写，即网络文件系统。Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保存的，同时NFS支持同时写操作。

emptyDir可以提供不同容器间的文件共享，但不能存储；hostPath可以为不同容器提供文件的共享并可以存储，但受制于节点限制，不能跨节点共享；这时需要网络存储 (NAS)，即既可以方便存储容器又可以从任何集群节点访问。

因为没有装NFS环境，本文略过，详细可参考 

* [NFS共享存储](https://blog.51cto.com/loong576/2435182#h17)
* [Kubernetes 之数据存储](https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&amp;mid=2247512338&amp;idx=4&amp;sn=04cc4697e9a84f193bd65a7275fefc18&amp;chksm=e918d40ede6f5d18c2df2c6743cf98ce6f4e91a9282eaec60b82c1ebca907a8554fecbd45320&amp;scene=178&amp;cur_album_id=1790241575034290179#rd)

中关于NFS的实验

## 5 PV & PVC

虽然 pod 也可以直接挂载存储，但是管理不方便，特别是 pod 的数量越来越多。而且 pod 可能是由开发维护的，而存储却是由运维负责。通过 PV、PVC 分开就方便多了。

- **PersistentVolume（PV）**

  是由管理员设置的存储，它是群集的一部分，用于描述一个具体的 Volume 属性，比如 Volume 的类型、挂载目录、远程存储服务器地址等。就像节点是集群中的资源一样，PV 也是集群中的资源。PV 是 Volume 之类的卷插件，但具有独立于使用 PV 的 Pod 的生命周期。此 API 对象包含存储实现的细节，即 NFS、iSCSI 或特定于云供应商的存储系统。

- **PersistentVolumeClaim（PVC）**

  是用户存储的请求，用于描述 Pod 想要使用的持久化属性，比如存储大小、读写权限等。它与 Pod 相似。Pod 消耗节点资源，PVC 消耗 PV 资源。Pod 可以请求特定级别的资源（CPU 和内存）。声明可以请求特定的大小和访问模式（例如，可以以读/写一次或只读多次模式挂载）。

- **StorageClass（SC）**

  充当 PV 的模板，自动为 PVC 创建 PV。

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/pvc.png" alt="pvc" style="zoom:50%;" />

当集群用户需要在其pod中使用持久化存储时，他们首先创建PVC清单，指定所需要的最低容量要求和访问模式，然后用户将待久卷声明清单提交给Kubernetes API服务器，Kubernetes将找到可匹配的PV并将其绑定到PVC。PVC可以当作pod中的一个卷来使用，其他用户不能使用相同的PV，除非先通过删除PVC绑定来释放。

![image-20210813151329994](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210813151329994.png)

## reference

https://blog.51cto.com/loong576/2435182

https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247512338&idx=4&sn=04cc4697e9a84f193bd65a7275fefc18&chksm=e918d40ede6f5d18c2df2c6743cf98ce6f4e91a9282eaec60b82c1ebca907a8554fecbd45320&scene=178&cur_album_id=1790241575034290179#rd

