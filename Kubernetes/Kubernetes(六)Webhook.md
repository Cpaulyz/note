# Kubernetes(六): Webhook

[TOC]



## 0 前言

本文参考博客https://xinchen.blog.csdn.net/article/details/113922328

基于前文的 elasticweb operator 进行开发

## 1 webhook介绍

### 1.1 是什么

* webhook类似java中的ServletFilter，外部对CRD资源的变更，在Controller处理之前都会交给webhook提前处理，流程如下图

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817164947977.png" alt="image-20210817164947977" style="zoom: 33%;" />

### 1.2 做什么

webhook可以做两件事

* **修改(mutating)**
* **验证(validating)**

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817165115932.png" alt="image-20210817165115932" style="zoom: 50%;" />

### 1.3 怎么用

* kubebuilder为我们提供了生成webhook的基础文件和代码的工具，与制作API的工具类似，极大地简化了工作量，咱们只需聚焦业务实现即可；
* 基于kubebuilder制作的webhook和controller，如果是同一个资源，那么它们在同一个进程中；

* 和controller类似，webhook既能在kubernetes环境中运行，也能在kubernetes环境之外运行；但如果webhook在kubernetes环境之外运行，是有些麻烦的，需要将证书放在所在环境，默认地址是：`/tmp/k8s-webhook-server/serving-certs/tls.{crt,key}`

## 2 环境搭建

- 为了更接近生产环境的用法，接下来的实战的做法是将webhook部署在kubernetes环境中
- 为了让webhook在kubernetes环境中运行，需要安装**cert manager**

```shell
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.2.0/cert-manager.yaml
```

- 上述操作完成后会新建很多资源，如namespace、rbac、pod等，以pod为例如下：

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817170406952.png" alt="image-20210817170406952" style="zoom:50%;" />

## 3 项目搭建

和api类似，进入elasticweb目录下，执行以下命令

```shell
kubebuilder create webhook --group webapp --version v1 --kind ElasticWeb --defaulting --programmatic-validation
```

![截屏2021-08-17 下午5.08.14](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8B%E5%8D%885.08.14.png)

在`elasticweb_webhook.go`中，有以下2处代码需要注意

* 第一个与**默认值**有关

![image-20210817171225516](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817171225516.png)

* 第二个与**校验**有关

![image-20210817171254286](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817171254286.png)

* 此外，需要在`config/default/kustomization.yaml`中打开以下注释

![截屏2021-08-17 下午5.14.28](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8B%E5%8D%885.14.28.png)

![截屏2021-08-17 下午5.17.18](https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/%E6%88%AA%E5%B1%8F2021-08-17%20%E4%B8%8B%E5%8D%885.17.18.png)

## 4 实战

### 4.1 需求

在前一章的elasticweb基础上增加以下功能

* 如果用户忘记输入totalQPS，系统webhook负责设置默认值1300
* 为了保护系统，单个pod的QPS设置上限1000，如果外部输入的singlePodQPS值超过1000，就创建资源对象失败

### 4.2 实现

只需修改`elastic_webhook.go`，在创建和更新时进行验证即可

````go
package v1

import (
	"k8s.io/apimachinery/pkg/api/errors"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/apimachinery/pkg/util/validation/field"
	ctrl "sigs.k8s.io/controller-runtime"
	logf "sigs.k8s.io/controller-runtime/pkg/log"
	"sigs.k8s.io/controller-runtime/pkg/webhook"
)
...

// ValidateCreate implements webhook.Validator so a webhook will be registered for the type
func (r *ElasticWeb) ValidateCreate() error {
	elasticweblog.Info("validate create", "name", r.Name)

	// TODO(user): fill in your validation logic upon object creation.
	return r.validateElasticWeb()
}

// ValidateUpdate implements webhook.Validator so a webhook will be registered for the type
func (r *ElasticWeb) ValidateUpdate(old runtime.Object) error {
	elasticweblog.Info("validate update", "name", r.Name)

	// TODO(user): fill in your validation logic upon object update.
	return r.validateElasticWeb()
}

func (r *ElasticWeb) validateElasticWeb() error{
	var allErrs field.ErrorList
	if *r.Spec.SinglePodQPS>1000{
		elasticweblog.Info("c. Invalid SinglePodQPS")
		err := field.Invalid(field.NewPath("spec").Child("singlePodQPS"),
			*r.Spec.SinglePodQPS,
			"d. must be less than 1000")

		allErrs = append(allErrs, err)

		return errors.NewInvalid(
			schema.GroupKind{Group: "webapp.com.cpaulyz", Kind: "ElasticWeb"},
			r.Name,
			allErrs)
	} else {
		elasticweblog.Info("e. SinglePodQPS is valid")
		return nil
	}
}
````

### 4.3 部署

* 部署CRD：`make install`

* 构建镜像并推送：`make docker-build docker-push IMG=cpaulyz/elasticweb:004          `

* 部署集成了webhook功能的controller：`make deploy IMG=cpaulyz/elasticweb:004`

* 查看pod，确认启动成功：

  ```shell
  chenyanze@chenyanzedeMacBook-Pro elasticweb % kubectl get pods -n elasticweb-system
  NAME                                             READY   STATUS    RESTARTS   AGE
  elasticweb-controller-manager-6b56c74b96-9mklw   2/2     Running   0          40s
  ```

### 4.4 运行验证

* 到`config/sample`中修改我们的yaml文件，假装我们忘记写totalQPS了，看看会怎么样

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    name: dev
---
apiVersion: webapp.com.cpaulyz/v1
kind: ElasticWeb
metadata:
  namespace: dev
  name: elasticweb-sample
spec:
  # Add fields here
  image: tomcat:8.0.18-jre8
  port: 30003
  singlePodQPS: 500
#  totalQPS: 3000
```

* 创建elatiscweb资源：` kubectl apply -f config/samples/webapp_v1_elasticweb.yaml `

* 可以看到，实际上创建了三个pod，并且spec里的totalQPS也是1300，说明已经填充了默认值（有一个pod一直在pending应该是因为资源受限了，这里不管）

<img src="https://cyzblog.oss-cn-beijing.aliyuncs.com/ali-mac/image-20210817182006832.png" alt="image-20210817182006832" style="zoom:50%;" />

* 再次修改yaml，把singlePodQPS调成非法值（大于1000）

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    name: dev
---
apiVersion: webapp.com.cpaulyz/v1
kind: ElasticWeb
metadata:
  namespace: dev
  name: elasticweb-sample
spec:
  # Add fields here
  image: tomcat:8.0.18-jre8
  port: 30003
  singlePodQPS: 1100
  totalQPS: 1200
```

* 运行后直接就会看到报错了

```shell
chenyanze@chenyanzedeMacBook-Pro elasticweb % kubectl apply -f config/samples/webapp_v1_elasticweb.yaml
namespace/dev unchanged
Error from server (ElasticWeb.webapp.com.cpaulyz "elasticweb-sample" is invalid: spec.singlePodQPS: Invalid value: 1100: d. must be less than 1000): error when applying patch:
{"metadata":{"annotations":{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"webapp.com.cpaulyz/v1\",\"kind\":\"ElasticWeb\",\"metadata\":{\"annotations\":{},\"name\":\"elasticweb-sample\",\"namespace\":\"dev\"},\"spec\":{\"image\":\"tomcat:8.0.18-jre8\",\"port\":30003,\"singlePodQPS\":1100,\"totalQPS\":1200}}\n"}},"spec":{"singlePodQPS":1100,"totalQPS":1200}}
to:
Resource: "webapp.com.cpaulyz/v1, Resource=elasticwebs", GroupVersionKind: "webapp.com.cpaulyz/v1, Kind=ElasticWeb"
Name: "elasticweb-sample", Namespace: "dev"
for: "config/samples/webapp_v1_elasticweb.yaml": admission webhook "velasticweb.kb.io" denied the request: ElasticWeb.webapp.com.cpaulyz "elasticweb-sample" is invalid: spec.singlePodQPS: Invalid value: 1100: d. must be less than 1000
```

### 4.5 清除环境

最后，别忘了清空环境

* 删除elasticweb资源对象：`kubectl delete -f config/samples/webapp_v1_elasticweb.yaml`
* 删除controller：`kustomize build config/default | kubectl delete -f -`
* 删除CRD：`make uninstall`