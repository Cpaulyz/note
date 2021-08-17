# Kubernetes(五): 使用Kuberbuilder搭建Operator

[TOC]

## 0 前言

本文内容主要参考教程 https://xinchen.blog.csdn.net/article/details/113035349

https://kubernetes.io/zh/docs/concepts/extend-kubernetes/operator/

对教程进行整理，仅供自己学习使用

注意⚠️：go需要自己手动下载1.15版本，brew install下载的会有一些小问题

## 1 项目搭建

### 环境搭建

https://xinchen.blog.csdn.net/article/details/113035349

### 项目构建

```shell
# 创建项目
mkdir -p $GOPATH/src/elasticweb
cd $GOPATH/src/elasticweb
kubebuilder init --domain com.cpaulyz
```

之后创建api

```shell
# 创建api，重点在于后面的参数设置
kubebuilder create api --group webapp --version v1 --kind ElasticWeb
```

这样一来我们就可以创建得到项目的框架了，其中框起来的是api相关的，其余是 `kubebuilder init `创建出来的，后面会对每个文件的内容进行介绍

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8A%E5%8D%8810.19.09.png" alt="截屏2021-08-17 上午10.19.09" style="zoom:50%;" />





## 2 项目结构介绍

operator工程新建完成后，会新增不少文件和目录，以下几个是官方提到的基础设施：

* go.mod：module的配置文件，里面已经填充了几个重要依赖；
* Makefile：非常重要的工具，前文咱们也用过了，编译构建、部署、运行都会用到；
* PROJECT：kubebuilder工程的元数据，在生成各种API的时候会用到这里面的信息；
* config/default：基于kustomize制作的配置文件，为controller提供标准配置，也可以按需要去修改调整；
* config/manager：一些和manager有关的细节配置，例如镜像的资源限制；
* config/rbac：顾名思义，如果像限制operator在kubernetes中的操作权限，就要通过rbac来做精细的权限配置了，这里面就是权限配置的细节；

> TODO

## 3 实战

### 3.1 需求

**背景**：

* QPS：Queries-per-second，既每秒查询率，就是说服务器在一秒的时间内处理了多少个请求；
* 背景：假设一个tomcat的QPS上限为500，如果外部访问的QPS达到了600，为了保障整个网站服务质量，必须再启动一个同样的tomcat来共同分摊请求
* 在kubernetes环境，如果外部请求超过了单个pod的处理极限，我们可以增加pod数量来达到横向扩容的目的

**任务**：

* 有一个springboot应用已经做成了docker镜像，我们要将其部署在k8s集群中
* 通过压测得出单个pod的QPS为500，估算得出上线后的总QPS会在800左右，随着运营策略变化，QPS还会有调整；
* 总的来说，我们手里只有三个数据：docker镜像、单个pod的QPS、总QPS。需要有个方案来帮我们将服务部署好，并且在运行期间能支撑外部的高并发访问；

**假设**：

* 假设每个pod所需的CPU和内存是固定的，直接在operator代码中写死，其实您也可以自己改代码，改成可以在外部配置，就像镜像名称参数那样

### 3.2 设计

#### CRD设计

**Spec部分：（用户期望值、终态）**

1. image：业务服务对应的镜像
2. port：service占用的宿主机端口，外部请求通过此端口访问pod的服务
3. singlePodQPS：单个pod的QPS上限
4. totalQPS：当前整个业务的总QPS

**Status部分：（实际值）**

realQPS，供 `kubectl describe` 命令查看当前可支持的QPS

#### 业务逻辑设计

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817151216564.png" alt="image-20210817151216564" style="zoom: 33%;" />

### 3.3 实现

#### CRD实现

修改api目录下的types.go文件即可

```go
package v1

import (
	"fmt"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"strconv"
)

// EDIT THIS FILE!  THIS IS SCAFFOLDING FOR YOU TO OWN!
// NOTE: json tags are required.  Any new fields you add must have json tags for the fields to be serialized.

// ElasticWebSpec defines the desired state of ElasticWeb
type ElasticWebSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	Image string `json:"image"`
	Port *int32 `json:"port"`
	SinglePodQPS *int32 `json:"single_pod_qps"`
	TotalQPS *int32 `json:"total_qps"`
}

// ElasticWebStatus defines the observed state of ElasticWeb
type ElasticWebStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	RealQPS *int32 `json:"real_qps"`
}

// +kubebuilder:object:root=true

// ElasticWeb is the Schema for the elasticwebs API
type ElasticWeb struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   ElasticWebSpec   `json:"spec,omitempty"`
	Status ElasticWebStatus `json:"status,omitempty"`
}

func (in *ElasticWeb) String() string {
	var realQPS string

	if nil == in.Status.RealQPS {
		realQPS = "nil"
	} else {
		realQPS = strconv.Itoa(int(*(in.Status.RealQPS)))
	}

	return fmt.Sprintf("Image [%s], Port [%d], SinglePodQPS [%d], TotalQPS [%d], RealQPS [%s]",
		in.Spec.Image,
		*(in.Spec.Port),
		*(in.Spec.SinglePodQPS),
		*(in.Spec.TotalQPS),
		realQPS)
}

// +kubebuilder:object:root=true

// ElasticWebList contains a list of ElasticWeb
type ElasticWebList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata,omitempty"`
	Items           []ElasticWeb `json:"items"`
}

func init() {
	SchemeBuilder.Register(&ElasticWeb{}, &ElasticWebList{})
}
```

### controller实现

```go
package controllers

import (
	"context"
	"fmt"
	"github.com/go-logr/logr"
	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	"k8s.io/apimachinery/pkg/api/resource"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/utils/pointer"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"

	webappv1 "elasticweb/api/v1"
)

// ElasticWebReconciler reconciles a ElasticWeb object
type ElasticWebReconciler struct {
	client.Client
	Log    logr.Logger
	Scheme *runtime.Scheme
}
const (
	// deployment中的APP标签名
	APP_NAME = "elastic-app"
	// tomcat容器的端口号
	CONTAINER_PORT = 8080
	// 单个POD的CPU资源申请
	CPU_REQUEST = "100m"
	// 单个POD的CPU资源上限
	CPU_LIMIT = "100m"
	// 单个POD的内存资源申请
	MEM_REQUEST = "512Mi"
	// 单个POD的内存资源上限
	MEM_LIMIT = "512Mi"
)

// 这里需要新增一个deployments和services的权限
// +kubebuilder:rbac:groups=webapp.com.cpaulyz,resources=elasticwebs,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=webapp.com.cpaulyz,resources=elasticwebs/status,verbs=get;update;patch
// +kubebuilder:rbac:groups=apps,resources=deployments,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=core,resources=services,verbs=get;list;watch;create;update;patch;delete

func (r *ElasticWebReconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
	ctx := context.Background()
	log := r.Log.WithValues("elasticweb", req.NamespacedName)
	// your logic here
	log.Info("1. start reconcile logic")
	// 实例化数据结构
	instance := &webappv1.ElasticWeb{}

	// 通过客户端工具查询，查询条件是
	err := r.Get(ctx, req.NamespacedName, instance)

	if err != nil {
		// 如果没有实例，就返回空结果，这样外部就不再立即调用Reconcile方法了
		if errors.IsNotFound(err) {
			log.Info("2.1. instance not found, maybe removed")
			return reconcile.Result{}, nil
		}
		log.Error(err, "2.2 error")
		// 返回错误信息给外部
		return ctrl.Result{}, err
	}
	log.Info("3. instance : " + instance.String())
	// 查找deployment
	deployment := &appsv1.Deployment{}
	// 用客户端工具查询
	err = r.Get(ctx, req.NamespacedName, deployment)
	// 查找时发生异常，以及查出来没有结果的处理逻辑
	if err != nil {
		// 如果没有实例就要创建了
		if errors.IsNotFound(err) {
			log.Info("4. deployment not exists")
			// 如果对QPS没有需求，此时又没有deployment，就啥事都不做了
			if *(instance.Spec.TotalQPS) < 1 {
				log.Info("5.1 not need deployment")
				// 返回
				return ctrl.Result{}, nil
			}
			// 先要创建service
			if err = createServiceIfNotExists(instance, r, ctx, req); err != nil {
				log.Error(err, "5.2 error")
				// 返回错误信息给外部
				return ctrl.Result{}, err
			}
			// 立即创建deployment
			if err = createDeploy(instance, r, ctx); err != nil {
				log.Error(err, "5.3 error")
				// 返回错误信息给外部
				return ctrl.Result{}, err
			}
			// 如果创建成功就更新状态
			if err = updateStatus(instance, r, ctx); err != nil {
				log.Error(err, "5.4. error")
				// 返回错误信息给外部
				return ctrl.Result{}, err
			}
			// 创建成功就可以返回了
			return ctrl.Result{}, nil
		} else {
			log.Error(err, "7. error")
			// 返回错误信息给外部
			return ctrl.Result{}, err
		}
	}
	// 如果查到了deployment，并且没有返回错误，就走下面的逻辑
	// 根据单QPS和总QPS计算期望的副本数
	expectReplicas := getExpectReplicas(instance)
	// 当前deployment的期望副本数
	realReplicas := *deployment.Spec.Replicas
	log.Info(fmt.Sprintf("9. expectReplicas [%d], realReplicas [%d]", expectReplicas, realReplicas))
	// 如果相等，就直接返回了
	if expectReplicas == realReplicas {
		log.Info("10. return now")
		return ctrl.Result{}, nil
	}
	// 如果不等，就要调整
	*(deployment.Spec.Replicas) = expectReplicas
	log.Info("11. update deployment's Replicas")
	// 通过客户端更新deployment
	if err = r.Update(ctx, deployment); err != nil {
		log.Error(err, "12. update deployment replicas error")
		// 返回错误信息给外部
		return ctrl.Result{}, err
	}
	log.Info("13. update status")
	// 如果更新deployment的Replicas成功，就更新状态
	if err = updateStatus(instance, r, ctx); err != nil {
		log.Error(err, "14. update status error")
		// 返回错误信息给外部
		return ctrl.Result{}, err
	}
	return ctrl.Result{}, nil
}

func (r *ElasticWebReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&webappv1.ElasticWeb{}).
		Complete(r)
}

// 计算所需要的副本数
func getExpectReplicas(web *webappv1.ElasticWeb) int32 {
	single := *(web.Spec.SinglePodQPS)
	total := *(web.Spec.TotalQPS)
	rep := total / single
	if total%single != 0 {
		rep++
	}
	return rep
}

// 创建 deployment
func createDeploy(web *webappv1.ElasticWeb, r *ElasticWebReconciler, ctx context.Context) error {
	log := r.Log.WithValues("func", "createDeploy")
	// 期望的副本数
	expectRep := getExpectReplicas(web)
	log.Info(fmt.Sprintf("Expect repilcac %d", expectRep))

	// 实例化一个数据结构
	deployment := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      web.Name,
			Namespace: web.Namespace,
		}, Spec: appsv1.DeploymentSpec{
			// 副本数是计算出来的
			Replicas: pointer.Int32Ptr(expectRep),
			Selector: &metav1.LabelSelector{
				MatchLabels: map[string]string{
					"app": APP_NAME,
				},
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: map[string]string{
						"app": APP_NAME,
					},
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{
						{
							Name: APP_NAME,
							// 用指定的镜像
							Image:           web.Spec.Image,
							ImagePullPolicy: "IfNotPresent",
							Ports: []corev1.ContainerPort{
								{
									Name:          "http",
									Protocol:      corev1.ProtocolSCTP,
									ContainerPort: CONTAINER_PORT,
								},
							},
							Resources: corev1.ResourceRequirements{
								Requests: corev1.ResourceList{
									"cpu":    resource.MustParse(CPU_REQUEST),
									"memory": resource.MustParse(MEM_REQUEST),
								},
								Limits: corev1.ResourceList{
									"cpu":    resource.MustParse(CPU_LIMIT),
									"memory": resource.MustParse(MEM_LIMIT),
								},
							},
						},
					},
				},
			},
		},
	}
	// 这一步非常关键！
	// 建立关联后，删除elasticweb资源时就会将deployment也删除掉
	log.Info("set reference")
	if err := controllerutil.SetControllerReference(web, deployment, r.Scheme); err != nil {
		log.Error(err, "SetControllerReference error")
		return err
	}
	// 创建deployment
	log.Info("start create deployment")
	if err := r.Create(ctx, deployment); err != nil {
		log.Error(err, "create deployment error")
		return err
	}
	log.Info("create deployment success")
	return nil
}

// 新建service
func createServiceIfNotExists(web *webappv1.ElasticWeb, r *ElasticWebReconciler, ctx context.Context, req ctrl.Request) error {
	log := r.Log.WithValues("func", "createService")

	service := &corev1.Service{}

	err := r.Get(ctx, req.NamespacedName, service)

	// 如果查询结果没有错误，证明service正常，就不做任何操作
	if err == nil {
		log.Info("service exists")
		return nil
	}

	// 如果错误不是NotFound，就返回错误
	if !errors.IsNotFound(err) {
		log.Error(err, "query service error")
		return err
	}

	// 实例化一个数据结构
	service = &corev1.Service{
		ObjectMeta: metav1.ObjectMeta{
			Namespace: web.Namespace,
			Name:      web.Name,
		},
		Spec: corev1.ServiceSpec{
			Ports: []corev1.ServicePort{{
				Name:     "http",
				Port:     8080,
				NodePort: *web.Spec.Port,
			},
			},
			Selector: map[string]string{
				"app": APP_NAME,
			},
			Type: corev1.ServiceTypeNodePort,
		},
	}

	// 这一步非常关键！
	// 建立关联后，删除elasticweb资源时就会将deployment也删除掉
	log.Info("set reference")
	if err := controllerutil.SetControllerReference(web, service, r.Scheme); err != nil {
		log.Error(err, "SetControllerReference error")
		return err
	}

	// 创建service
	log.Info("start create service")
	if err := r.Create(ctx, service); err != nil {
		log.Error(err, "create service error")
		return err
	}

	log.Info("create service success")

	return nil
}

// 更新status，使得外部可以通过 describe 命令来查看当前的状态
func updateStatus(web *webappv1.ElasticWeb, r *ElasticWebReconciler, ctx context.Context) error {
	log := r.Log.WithValues("func", "updateStatus")

	// 单个pod的QPS
	singlePodQPS := *(web.Spec.SinglePodQPS)

	// pod总数
	replicas := getExpectReplicas(web)

	// 当pod创建完毕后，当前系统实际的QPS：单个pod的QPS * pod总数
	// 如果该字段还没有初始化，就先做初始化
	if nil == web.Status.RealQPS {
		web.Status.RealQPS = new(int32)
	}

	*(web.Status.RealQPS) = singlePodQPS * replicas

	log.Info(fmt.Sprintf("singlePodQPS [%d], replicas [%d], realQPS[%d]", singlePodQPS, replicas, *(web.Status.RealQPS)))

	if err := r.Update(ctx, web); err != nil {
		log.Error(err, "update instance error")
		return err
	}
	return nil
}
```

### 3.4 部署

operator编写完以后，要把它启动起来。可以将operator理解为一个服务，可以本地部署、也可以打包成镜像，部署到k8s中（有点套娃的意思）

#### 本地运行

首先使用本地运行的方式，可以理解为另外起一个服务，步骤如下：

* 直接 `make install` 

![image-20210817113146347](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817113146347.png)

* `make run`

![image-20210817113249990](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817113249990.png)

* 完成

#### 镜像部署

* 先在hub.docker.com中注册账号，然后在命令行输入docker login登陆
* 执行命令 `make docker-build docker-push IMG=cpaulyz/elasticweb:001`，打包成镜像推送到dockerhub中，镜像名为cpaulyz/elasticweb，tag为001

![image-20210817114828323](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817114828323.png)

* 然后就可以在网页上看到镜像啦

![image-20210817114730844](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817114730844.png)

* 执行 `make deploy IMG=cpaulyz/elasticweb:001` 即可在部署在k8s中，可以进行查看一下pods，注意这里的namespace不一样

![截屏2021-08-17 上午11.52.25](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8A%E5%8D%8811.52.25.png)

### 3.5 验证

通过运行一些 CR(customer resource) 相关的请求，来验证部署后的的operator功能吧

* 首先新建elasticweb资源对象，在config/samples目录下，kubebuilder为咱们创建了demo文件elasticweb_v1_elasticweb.yaml，不过这里面spec的内容不是咱们定义的那四个字段，需要改成以下内容

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    name: dev
---
apiVersion: webapp.com.cpaulyz/v1 # 与 kubectl api-versions 查到的一致
kind: ElasticWeb
metadata:
  namespace: dev
  name: elasticweb-sample
spec:
  # Add fields here
  image: tomcat:8.0.18-jre8
  port: 30003
  singlePodQPS: 500
  totalQPS: 600
```

* 运行 `kubectl apply -f config/samples/webapp_v1_elasticweb.yamlp`，可以看到我们需要的东西已经被创建出来了

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817142854373.png" alt="image-20210817142854373" style="zoom:50%;" />

* 访问service，这里我是用minikube搭建的集群环境，使用minikube进行访问，也可以直接通过nodePort进行访问

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817143127579.png" alt="image-20210817143127579" style="zoom:50%;" />

* 如果是通过本地运行的，可以直接在控制台看到日志输出；如果是镜像部署的，需要进入到pod的容器中进行查看。

  * 首先先查看pod信息，`kubectl get pods -n elasticweb-system`

    可以看到有两个contrainer

    ![image-20210817145623317](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817145623317.png)

  * describe查看详情，通过镜像不难看出我们要的是maneger这个容器

    ![截屏2021-08-17 下午2.57.16](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8B%E5%8D%882.57.16.png)

  * 查看日志，`kubectl logs -f elasticweb-controller-manager-56ff5c985d-gdvm6 -c manager -n elasticweb-system`

    ![image-20210817145908990](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817145908990.png)

  * 非常好！说明部署得很成功！

* 接下来，我们验证修改。将yaml中的totalQPS改为3000，然后再次 `kubectl apply -f config/samples/webapp_v1_elasticweb.yaml `，可以看到新的pod准备启动了。

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817150620795.png" alt="image-20210817150620795" style="zoom:50%;" />

* 再来看看status字段，`kubectl describe elasticwebs -n dev`

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817150740814.png" alt="image-20210817150740814" style="zoom:50%;" />

* 最后测试删除，`kubectl delete elasticwebs elasticweb-sample -n dev`，可以看到所有的相关资源都被自动释放了。

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817151007996.png" alt="image-20210817151007996" style="zoom:50%;" />