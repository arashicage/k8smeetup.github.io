---
title: 使用 Helm 安装服务目录（Service Catalog）
approvers:
- chenopis
cn-approvers:
- lichuqiang
---


{% capture overview %}


为使用 [Helm](https://helm.sh/) 在 Kubernetes 集群中安装服务目录，用户可以在 [kubernetes-incubator/service-catalog](https://github.com/kubernetes-incubator/service-catalog/blob/master/docs/install.md) 仓库中找到相关流程的最新信息。

{% endcapture %}


{% capture prerequisites %}

* 理解 [服务目录](/docs/concepts/service-catalog/) 的关键概念。

* 服务目录要求 Kubernetes 集群为 1.7 或更高版本。

* 您必须在集群中启用集群 DNS。
    * 如果您使用的是基于云服务的 Kubernetes 集群或 Minikube，您可能已经启用了群集 DNS。
    * 如果您使用 `hack/local-up-cluster.sh`，请确保先设置 `KUBE_ENABLE_CLUSTER_DNS` 环境变量，
    然后再运行安装脚本。

* [安装并设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)（1.7 或更高版本），
确保将其配置为连接到 Kubernetes 集群。

* 安装 2.7.0 或更新版本的 [Helm](http://helm.sh/) 。
    * 按照 [Helm 安装说明](https://github.com/kubernetes/helm/blob/master/docs/install.md) 进行安装。
    * 如果您已经安装了合适的 Helm 版本，那么执行 `helm init` 来安装 Tiller，它是 Helm 的服务器端组件。

{% endcapture %}


{% capture steps %}

## 添加服务目录 Helm 仓库

一旦 Helm 安装完成， 通过执行以下命令，在您的本地机器上添加 *service-catalog* Helm 仓库：

```shell
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```


通过执行以下命令进行检查，确保其安装成功安装：

```shell
helm search service-catalog
```


如果安装成功，命令应输出以下内容：

```
NAME            VERSION DESCRIPTION
svc-cat/catalog 0.0.1   service-catalog API server and controller-manag...
```


## 启用 RBAC

您的 Kubernetes 集群须启用 RBAC， 这要求您的 Tiller Pod(s) 拥有 `cluster-admin` 访问权限。


如果您正使用 Minikube，运行 `minikube start` 命令时加入以下参数：

```shell
minikube start --extra-config=apiserver.Authorization.Mode=RBAC
```


如果您正使用 `hack/local-up-cluster.sh`，将 `AUTHORIZATION_MODE` 环境变量设置为以下值：

```
AUTHORIZATION_MODE=Node,RBAC hack/local-up-cluster.sh -O
```


默认情况下， `helm init` 将 Tiller Pod 安装到 `kube-system` 名字空间下，并配置 Tiller 使用 `default`
service account。


**注意：** 如果您在运行 `helm init` 时使用了 `--tiller-namespace` 或 `--service-account` 参数，
那么您需要适当地调整下面命令中的 `--serviceaccount` 参数，令其引用相应的名字空间和 ServiceAccount 名称。
{: .note}


配置 Tiller ，使其具有 `cluster-admin` 访问权限：

```shell
kubectl create clusterrolebinding tiller-cluster-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:default
```



## 在 Kubernetes 集群中安装服务目录

从 Helm 仓库的根目录，使用以下命令安装服务目录：

```shell
helm install svc-cat/catalog \
    --name catalog --namespace catalog
```

{% endcapture %}


{% capture whatsnext %}

* 查看 [服务代理（service broker）样例](https://github.com/openservicebrokerapi/servicebroker/blob/master/gettingStarted.md#sample-service-brokers)。
* 探索 [kubernetes-incubator/service-catalog](https://github.com/kubernetes-incubator/service-catalog) 项目。

{% endcapture %}


{% include templates/task.md %}
