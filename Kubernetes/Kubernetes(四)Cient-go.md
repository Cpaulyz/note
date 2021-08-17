# Kubernetes(四): Cient-go Demo

[TOC]

转载+整理自：https://xinchen.blog.csdn.net/article/details/113035349

## 0. 前言

学习k8s的时候，绕不开使用client-go。

在网上看到了一个非常不错的教程 https://xinchen.blog.csdn.net/article/details/113035349

对其中的demo部分进行了整理，仅作学习使用。

环境：

![image-20210816161350646](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816161350646.png)

## 1. RestClient

### 1.1 简介

RESTClient是client-go最基础的客户端，主要是对HTTP Reqeust进行了封装，对外提供RESTful风格的API，并且提供丰富的API用于各种设置，相比其他几种客户端虽然更复杂，但是也更为灵活；

使用RESTClient对kubernetes的资源进行增删改查的基本步骤如下：

1. 确定要操作的资源类型(例如查找deployment列表)，去官方API文档中找到对于的path、数据结构等信息，后面会用到；
2. 加载配置kubernetes配置文件（和kubectl使用的那种kubeconfig完全相同）；
3. 根据配置文件生成配置对象，并且通过API对配置对象就行设置（例如请求的path、Group、Version、序列化反序列化工具等）；
   创建RESTClient实例，入参是配置对象；
4. 调用RESTClient实例的方法向kubernetes的API Server发起请求，编码用fluent风格将各种参数传入(例如指定namespace、资源等)，如果是查询类请求，还要传入数据结构实例的指针，改数据结构用于接受kubernetes返回的查询结果；

### 1.2 实战

> 任务：查询kube-system这个namespace下的所有pod，然后在控制台打印每个pod的几个关键字段
>
> 文档地址：https://v1-19.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/

* 新建文件夹并引入依赖

```shell
chenyanze@chenyanzedeMacBook-Pro client-go % mkdir rest-client-demo
chenyanze@chenyanzedeMacBook-Pro client-go % cd rest-client-demo 
chenyanze@chenyanzedeMacBook-Pro rest-client-demo % go mod init rest-client-demo
go: creating new go.mod: module rest-client-demo
chenyanze@chenyanzedeMacBook-Pro rest-client-demo % go get k8s.io/api@v0.20.0
go: downloading k8s.io/api v0.20.0
go build k8s.io/api: no non-test Go files in /Users/chenyanze/projects/GolandProjects/pkg/mod/k8s.io/api@v0.20.0
chenyanze@chenyanzedeMacBook-Pro rest-client-demo % go get k8s.io/client-go@v0.20.0
go: downloading k8s.io/client-go v0.20.0
```

> ⚠️注意：这里的版本和kubernetes版本相关，关于kubernetes版本可以通过命令`kubectl version`进行查看，对应版本信息可参照官方仓库的相关文档https://github.com/kubernetes/client-go

* 运行代码

```go
package main

import (
	"context"
	"flag"
	"fmt"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes/scheme"
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/util/homedir"
	"path/filepath"
)

func main() {
	var kubeconfig *string

	// home是家目录，如果能取得家目录的值，就可以用来做默认值
	if home := homedir.HomeDir(); home != "" {
		// 如果输入了kubeconfig参数，该参数的值就是kubeconfig文件的绝对路径，
		// 如果没有输入kubeconfig参数，就用默认路径~/.kube/config
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		// 如果取不到当前用户的家目录，就没办法设置kubeconfig的默认目录了，只能从入参中取
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}

	flag.Parse()

	// 从本机加载kubeconfig配置文件，因此第一个参数为空字符串
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)

	// kubeconfig加载失败就直接退出了
	if err != nil {
		panic(err.Error())
	}

	// 参考path : /api/v1/namespaces/{namespace}/pods
	config.APIPath = "api"
	// pod的group是空字符串
	config.GroupVersion = &corev1.SchemeGroupVersion
	// 指定序列化工具
	config.NegotiatedSerializer = scheme.Codecs

	// 根据配置信息构建restClient实例
	restClient, err := rest.RESTClientFor(config)

	if err != nil {
		panic(err.Error())
	}

	// 保存pod结果的数据结构实例
	result := &corev1.PodList{}

	//  指定namespace
	namespace := "kube-system"
	// 设置请求参数，然后发起请求
	// GET请求
	err = restClient.Get().
		//  指定namespace，参考path : /api/v1/namespaces/{namespace}/pods
		Namespace(namespace).
		// 查找多个pod，参考path : /api/v1/namespaces/{namespace}/pods
		Resource("pods").
		// 指定大小限制和序列化工具
		VersionedParams(&metav1.ListOptions{Limit: 100}, scheme.ParameterCodec).
		// 请求
		Do(context.TODO()).
		// 结果存入result
		Into(result)

	if err != nil {
		panic(err.Error())
	}

	// 表头
	fmt.Printf("namespace\t status\t\t name\n")

	// 每个pod都打印namespace、status.Phase、name三个字段
	for _, d := range result.Items {
		fmt.Printf("%v\t %v\t %v\n",
			d.Namespace,
			d.Status.Phase,
			d.Name)
	}
}
```

* 可以看到结果

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816143644795.png" alt="image-20210816143644795" style="zoom:50%;" />

## 2. ClientSet

### 2.1 简介

* 前文学习了最基础的客户端Restclient，尽管咱们实战的需求很简单（获取指定namespace下所有pod的信息），但还是写了不少代码，如下图，各种设置太麻烦，例如api的path、Group、Version、返回的数据结构、编解码工具等
* ClientSet是对RestClient的一个封装
* 具体的源码解析可以参考https://xinchen.blog.csdn.net/article/details/113788269，由于本文只作demo记录，不再展开。

### 2.2 实战

> * 写一段代码，检查用户输入的operate参数，该参数默认是create，也可以接受clean；
>   * 如果operate参数等于create，就执行以下操作：
>     * 新建名为test-clientset的**namespace**
>     * 新建一个**deployment**，namespace为test-clientset，镜像用tomcat，副本数为2
>     * 新建一个**service**，namespace为test-clientset，类型是NodePort
>   * 如果operate参数等于clean，就删除create操作中创建的service、deployment、namespace等资源：
> * 以上需求使用Clientset客户端实现，完成后咱们用浏览器访问来验证tomcat是否正常；

* 创建文件夹并添加依赖

```shell
chenyanze@chenyanzedeMacBook-Pro client-go % mkdir client-set-demo
chenyanze@chenyanzedeMacBook-Pro client-go % cd client-set-demo 
chenyanze@chenyanzedeMacBook-Pro client-set-demo % go mod init client-set-demo
go: creating new go.mod: module client-set-demo
chenyanze@chenyanzedeMacBook-Pro client-set-demo % go get k8s.io/api@v0.20.0
chenyanze@chenyanzedeMacBook-Pro client-set-demo % go get k8s.io/client-go@v0.20.0
```

* 代码如下

```go
package main

import (
	"context"
	"flag"
	"fmt"
	appsv1 "k8s.io/api/apps/v1"
	apiv1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/util/homedir"
	"k8s.io/utils/pointer"
	"path/filepath"
)

const (
	NAMESPACE = "test-clientset"
	DEPLOYMENT_NAME = "client-test-deployment"
	SERVICE_NAME = "client-test-service"
)

func main() {
	var kubeconfig *string

	// home是家目录，如果能取得家目录的值，就可以用来做默认值
	if home:=homedir.HomeDir(); home != "" {
		// 如果输入了kubeconfig参数，该参数的值就是kubeconfig文件的绝对路径，
		// 如果没有输入kubeconfig参数，就用默认路径~/.kube/config
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		// 如果取不到当前用户的家目录，就没办法设置kubeconfig的默认目录了，只能从入参中取
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}
	// 获取用户输入的操作类型，默认是create，还可以输入clean，用于清理所有资源
	operate := flag.String("operate", "create", "operate type : create or clean")
	flag.Parse()
	// 从本机加载kubeconfig配置文件，因此第一个参数为空字符串
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	// kubeconfig加载失败就直接退出了
	if err != nil {
		panic(err.Error())
	}

	// 实例化clientset对象
	clientset, err := kubernetes.NewForConfig(config)
	if err!= nil {
		panic(err.Error())
	}

	fmt.Printf("operation is %v\n", *operate)

	// 如果要执行清理操作
	if "clean"==*operate {
		clean(clientset)
	} else {
		// 创建namespace
		createNamespace(clientset)
		// 创建deployment
		createDeployment(clientset)
		// 创建service
		createService(clientset)
	}
}

// 清理本次实战创建的所有资源
func clean(clientset *kubernetes.Clientset) {
	emptyDeleteOptions := metav1.DeleteOptions{}
	// 删除service
	if err := clientset.CoreV1().Services(NAMESPACE).Delete(context.TODO(), SERVICE_NAME, emptyDeleteOptions) ; err != nil {
		panic(err.Error())
	}
	// 删除deployment
	if err := clientset.AppsV1().Deployments(NAMESPACE).Delete(context.TODO(), DEPLOYMENT_NAME, emptyDeleteOptions) ; err != nil {
		panic(err.Error())
	}
	// 删除namespace
	if err := clientset.CoreV1().Namespaces().Delete(context.TODO(), NAMESPACE, emptyDeleteOptions) ; err != nil {
		panic(err.Error())
	}
}

// 新建namespace
func createNamespace(clientset *kubernetes.Clientset) {
	namespaceClient := clientset.CoreV1().Namespaces()
	namespace := &apiv1.Namespace{
		ObjectMeta: metav1.ObjectMeta{
			Name: NAMESPACE,
		},
	}
	result, err := namespaceClient.Create(context.TODO(), namespace, metav1.CreateOptions{})
	if err!=nil {
		panic(err.Error())
	}
	fmt.Printf("Create namespace %s \n", result.GetName())
}

// 新建service
func createService(clientset *kubernetes.Clientset) {
	// 得到service的客户端
	serviceClient := clientset.CoreV1().Services(NAMESPACE)

	// 实例化一个数据结构
	service := &apiv1.Service{
		ObjectMeta: metav1.ObjectMeta{
			Name: SERVICE_NAME,
		},
		Spec: apiv1.ServiceSpec{
			Ports: []apiv1.ServicePort{{
				Name: "http",
				Port: 8080,
				NodePort: 30080,
			},
			},
			Selector: map[string]string{
				"app" : "tomcat",
			},
			Type: apiv1.ServiceTypeNodePort,
		},
	}

	result, err := serviceClient.Create(context.TODO(), service, metav1.CreateOptions{})

	if err!=nil {
		panic(err.Error())
	}

	fmt.Printf("Create service %s \n", result.GetName())
}

// 新建deployment
func createDeployment(clientset *kubernetes.Clientset) {
	// 得到deployment的客户端
	deploymentClient := clientset.
		AppsV1().
		Deployments(NAMESPACE)

	// 实例化一个数据结构
	deployment := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name: DEPLOYMENT_NAME,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: pointer.Int32Ptr(2),
			Selector: &metav1.LabelSelector{
				MatchLabels: map[string]string{
					"app" : "tomcat",
				},
			},

			Template: apiv1.PodTemplateSpec{
				ObjectMeta:metav1.ObjectMeta{
					Labels: map[string]string{
						"app" : "tomcat",
					},
				},
				Spec: apiv1.PodSpec{
					Containers: []apiv1.Container{
						{
							Name: "tomcat",
							Image: "tomcat:8.0.18-jre8",
							ImagePullPolicy: "IfNotPresent",
							Ports: []apiv1.ContainerPort{
								{
									Name: "http",
									Protocol: apiv1.ProtocolSCTP,
									ContainerPort: 8080,
								},
							},
						},
					},
				},
			},
		},
	}

	result, err := deploymentClient.Create(context.TODO(), deployment, metav1.CreateOptions{})

	if err!=nil {
		panic(err.Error())
	}

	fmt.Printf("Create deployment %s \n", result.GetName())
}
```

* 运行 `go run main.go`，可以看到pods和service已经都起起来了

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816150212133.png" alt="image-20210816150212133" style="zoom:50%;" />

* 等待ready以后，进行一波访问即可

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816151827469.png" alt="image-20210816151827469" style="zoom:33%;" />

* 执行命令 `go run main.go -operate clean`即可删除刚才创建的所有资源

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816152006486.png" alt="image-20210816152006486" style="zoom:50%;" />

### 2.3 技巧

使用分屏，对应着编写资源的数据结构

![20210212143832594](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/20210212143832594.png)

## 3. DynamicClient

> 注意⚠️：本文涉及到关于Object.runtime和Unstructed对象，可以参考https://xinchen.blog.csdn.net/article/details/113795523

### 3.1 简介

1. 与Clientset不同，dynamicClient为各种类型的资源都提供统一的操作API，资源需要包装为Unstructured数据结构；
2. 内部使用了Restclient与kubernetes交互；

### 3.2 实战

> * 本次编码实战的需求很简单：查询指定namespace下的所有pod，然后在控制台打印出来，要求用dynamicClient实现；
> * 您可能会问：pod是kubernetes的内置资源，更适合Clientset来操作，而dynamicClient更适合处理CRD不是么？—您说得没错，这里用pod是因为折腾CRD太麻烦了，定义好了还要在kubernetes上发布，于是干脆用pod来代替CRD，反正dynamicClient都能处理，咱们通过实战掌握dynamicClient的用法就行了，以后遇到各种资源都能处理之；

* 创建文件夹并引入依赖

```shell
chenyanze@chenyanzedeMacBook-Pro client-go % mkdir dynamic-client-demo
chenyanze@chenyanzedeMacBook-Pro client-go % cd dynamic-client-demo 
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go mod init dynamic-client-demo
go: creating new go.mod: module dynamic-client-demo
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go get k8s.io/api@v0.20.0
go build k8s.io/api: no non-test Go files in /Users/chenyanze/projects/GolandProjects/pkg/mod/k8s.io/api@v0.20.0
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go get k8s.io/client-go@v0.20.0
```

* 编写代码

```go
package main

import (
	"context"
	"flag"
	"fmt"
	apiv1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/client-go/dynamic"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/util/homedir"
	"path/filepath"
)

func main() {

	var kubeconfig *string

	// home是家目录，如果能取得家目录的值，就可以用来做默认值
	if home:=homedir.HomeDir(); home != "" {
		// 如果输入了kubeconfig参数，该参数的值就是kubeconfig文件的绝对路径，
		// 如果没有输入kubeconfig参数，就用默认路径~/.kube/config
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		// 如果取不到当前用户的家目录，就没办法设置kubeconfig的默认目录了，只能从入参中取
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}

	flag.Parse()

	// 从本机加载kubeconfig配置文件，因此第一个参数为空字符串
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)

	// kubeconfig加载失败就直接退出了
	if err != nil {
		panic(err.Error())
	}

	dynamicClient, err := dynamic.NewForConfig(config)

	if err != nil {
		panic(err.Error())
	}

	// dynamicClient的唯一关联方法所需的入参
	gvr := schema.GroupVersionResource{Version: "v1", Resource: "pods"}

	// 使用dynamicClient的查询列表方法，查询指定namespace下的所有pod，
	// 注意此方法返回的数据结构类型是UnstructuredList
	unstructObj, err := dynamicClient.
		Resource(gvr).
		Namespace("kube-system").
		List(context.TODO(), metav1.ListOptions{Limit: 100})

	if err != nil {
		panic(err.Error())
	}

	// 实例化一个PodList数据结构，用于接收从unstructObj转换后的结果
	podList := &apiv1.PodList{}

	// 转换
	err = runtime.DefaultUnstructuredConverter.FromUnstructured(unstructObj.UnstructuredContent(), podList)

	if err != nil {
		panic(err.Error())
	}

	// 表头
	fmt.Printf("namespace\t status\t\t name\n")

	// 每个pod都打印namespace、status.Phase、name三个字段
	for _, d := range podList.Items {
		fmt.Printf("%v\t %v\t %v\n",
			d.Namespace,
			d.Status.Phase,
			d.Name)
	}
}
```

* 运行

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816155325736.png" alt="image-20210816155325736" style="zoom:50%;" />

## 4. DiscoveryClient

###  4.1 简介

* 咱们之前学习的Clientset和dynamicClient都是面向资源对象的（例如创建deployment实例、查看pod实例）
* DiscoveryClient则不同，它聚焦的是资源，例如查看当前kubernetes有哪些Group、Version、Resource

> Pod、Deployment等叫做资源，创建一个Pod，创建出来的东西叫做资源对象

### 4.2 实战

> 从kubernetes查询所有的Group、Version、Resource信息，在控制台打印出来

* 创建文件夹并引入依赖

```shell
chenyanze@chenyanzedeMacBook-Pro client-go % mkdir dynamic-client-demo
chenyanze@chenyanzedeMacBook-Pro client-go % cd dynamic-client-demo 
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go mod init dynamic-client-demo
go: creating new go.mod: module dynamic-client-demo
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go get k8s.io/client-go@v0.20.0
chenyanze@chenyanzedeMacBook-Pro dynamic-client-demo % go get k8s.io/api@v0.20.0 
```

* 编写代码

```go
package main

import (
	"flag"
	"fmt"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/client-go/discovery"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/util/homedir"
	"path/filepath"
)

func main() {

	var kubeconfig *string

	// home是家目录，如果能取得家目录的值，就可以用来做默认值
	if home:=homedir.HomeDir(); home != "" {
		// 如果输入了kubeconfig参数，该参数的值就是kubeconfig文件的绝对路径，
		// 如果没有输入kubeconfig参数，就用默认路径~/.kube/config
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		// 如果取不到当前用户的家目录，就没办法设置kubeconfig的默认目录了，只能从入参中取
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}

	flag.Parse()

	// 从本机加载kubeconfig配置文件，因此第一个参数为空字符串
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	// kubeconfig加载失败就直接退出了
	if err != nil {
		panic(err.Error())
	}

	// 新建discoveryClient实例
	discoveryClient, err := discovery.NewDiscoveryClientForConfig(config)
	if err != nil {
		panic(err.Error())
	}

	// 获取所有分组和资源数据
	APIGroup, APIResourceListSlice, err := discoveryClient.ServerGroupsAndResources()
	if err != nil {
		panic(err.Error())
	}

	// 先看Group信息
	fmt.Printf("APIGroup :\n\n %v\n\n\n\n",APIGroup)

	// APIResourceListSlice是个切片，里面的每个元素代表一个GroupVersion及其资源
	for _, singleAPIResourceList := range APIResourceListSlice {

		// GroupVersion是个字符串，例如"apps/v1"
		groupVerionStr := singleAPIResourceList.GroupVersion

		// ParseGroupVersion方法将字符串转成数据结构
		gv, err := schema.ParseGroupVersion(groupVerionStr)

		if err != nil {
			panic(err.Error())
		}

		fmt.Println("*****************************************************************")
		fmt.Printf("GV string [%v]\nGV struct [%#v]\nresources :\n", groupVerionStr, gv)

		// APIResources字段是个切片，里面是当前GroupVersion下的所有资源
		for _, singleAPIResource := range singleAPIResourceList.APIResources {
			fmt.Printf("%v\n", singleAPIResource.Name)
		}
	}
}
```

* 运行结果（节选）

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210816160957135.png" alt="image-20210816160957135" style="zoom:50%;" />

> 事实上，`kubectl api-resources` 底层也是调用DiscoveryClient来进行实现的

