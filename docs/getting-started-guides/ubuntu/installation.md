---
approvers:
- caesarxuchao
- erictune
title: Setting up Kubernetes with Juju
---



{% capture overview %}
Ubuntu 16.04 已公开 [Kubernetes 的 Canonical 发行版 ](https://www.ubuntu.com/cloud/kubernetes), 一套为生产环境设计的 Kubernetes 上游版本。本文将为您演示如何部署集群。
{% endcapture %}



{% capture prerequisites %}
- 一个运行的 [Juju 客户端](https://jujucharms.com/docs/2.2/reference-install)；可以是一台 Linux，也可以是 Windows 或 OSX。
- 一个 [受支持的云](#cloud-compatibility)。
  - 通过 [MAAS](http://maas.io)  实现裸机部署。 配置指南参考 [MAAS 文档](http://maas.io/docs/)。
  - 目前只在 OpenStack 的 Icehouse 及以上版本测试过。
- 对以下域的网络访问
  - *.jujucharms.com
  - gcr.io
  - github.com
  - 访问 Ubuntu 镜像源（公有或私有）
{% endcapture %}



{% capture steps %}
## 部署概述
开箱即用的部署可以在 9 台服务器上完成以下的组件：

- Kubernetes （自动地部署，操作及扩展）
     - 具有一个主节点和两个工作节点的三节点 Kubernetes 集群。
     - 启用 TLS 用于单元之间的安全通信。
     - Flannel 软件定义网络 (SDN) 插件
     - 一个负载均衡器以实现 kubernetes-master 的高可用 (实验阶段)
     - 可选的 Ingress Controller （在工作节点）
     - 可选的 Dashboard 插件 （在主节点）， 包含实现集群监控的 Heapster
- EasyRSA
     - 扮演一个证书颁发机构的角色，可以提供自签名证书
- ETCD （分布式键值存储）
     - 三节点的高可用集群。



Juju Kubernetes 项目由 Canonical Ltd（https://www.canonical.com/） 的 Big Software 团队策划，您可以获知我们是如何做的。如果您发现任何问题，请打开 [issue on our tracker](https://github.com/juju-solutions/bundle-canonical-kubernetes)，以便我们解决。



##  支持级别

IaaS 提供商        | 配置管理 | 系统     | 网络  | 文档                                              | 符合 | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
Amazon Web Services (AWS)   | Juju         | Ubuntu | flannel, calico*     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
OpenStack                   | Juju         | Ubuntu | flannel, calico     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
Microsoft Azure             | Juju         | Ubuntu | flannel     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
Google Compute Engine (GCE) | Juju         | Ubuntu | flannel, calico     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
Joyent                      | Juju         | Ubuntu | flannel     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
Rackspace                   | Juju         | Ubuntu | flannel     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
VMWare vSphere              | Juju         | Ubuntu | flannel, calico     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)
Bare Metal (MAAS)           | Juju         | Ubuntu | flannel, calico     | [docs](/docs/getting-started-guides/ubuntu)                                   |          | [Commercial](https://ubuntu.com/cloud/kubernetes), [Community](https://github.com/juju-solutions/bundle-kubernetes-core)



有关所有解决方案的支持级别信息，请参见 [解决方案表](/docs/getting-started-guides/#table-of-solutions)。

## 配置 Juju 以便使用您的 cloud provider

可以在 [各种公有云](#cloud-compatibility)，私有 OpenStack 云，或者是原始的裸机集群上部署集群软件。通过 [MAAS](http://maas.io)  实现裸机部署。

确定部署在哪类云上后，按照 [云安装界面](https://jujucharms.com/docs/devel/getting-started) 配置部署到该云。

为您使用的 cloud provider 加载 [云凭证](https://jujucharms.com/docs/2.2/credentials)



在本例中

```
juju add-credential aws
credential name: my_credentials
select auth-type [userpass, oauth, etc]: userpass
enter username: jorge
enter password: *******
```

您也可以使用 `juju autoload-credentials` 命令自动加载常用的云凭证，该命令将自动从每个云的默认文件和环境变量中导入凭据。



接下来，我们需要引导一个 controller 来管理集群。您需要定义要引导的云，区域以及任意一个 controller 节点名：

```
juju update-clouds # 这个命令可以确保客户端上所有最新的区域是最新的
juju bootstrap aws/us-east-2
```
或者，另外一个例子，这次在 Azure 上：

```
juju bootstrap azure/centralus
```



您需要为部署到的每个云或区域分配一个 controller 节点。有关更多信息，请参阅[controller文档](https://jujucharms.com/docs/2.2/controllers)。

请注意，每个 controller 可以在给定的云或区域中托管多个 Kubernetes 集群。



## 启动 Kubernetes 集群

以下命令将部署 9 节点的初始启动集群。执行速度取决于您要部署到的云的性能：

```
juju deploy canonical-kubernetes
```

执行此命令后，云将启动实例并开始部署过程。



## 监控部署

`juju status` 命令提供集群中每个单元的信息。`watch -c juju status --color`命令可以获取集群部署的实时状态。当所有的状态是绿色的并且“空闲”时，表示集群已准备好：

juju status

Output:

```
Model    Controller     Cloud/Region   Version
default  aws-us-east-2  aws/us-east-2  2.0.1

App                    Version  Status       Scale  Charm                  Store       Rev  OS      Notes
easyrsa                3.0.1    active           1  easyrsa                jujucharms    3  ubuntu  
etcd                   3.1.2    active           3  etcd                   jujucharms   14  ubuntu  
flannel                0.6.1    maintenance      4  flannel                jujucharms    5  ubuntu  
kubeapi-load-balancer  1.10.0   active           1  kubeapi-load-balancer  jujucharms    3  ubuntu  exposed
kubernetes-master      1.6.1    active           1  kubernetes-master      jujucharms    6  ubuntu  
kubernetes-worker      1.6.1    active           3  kubernetes-worker      jujucharms    8  ubuntu  exposed
topbeat                         active           3  topbeat                jujucharms    5  ubuntu  

Unit                      Workload     Agent  Machine  Public address  Ports            Message
easyrsa/0*                active       idle   0        52.15.95.92                      Certificate Authority connected.
etcd/0                    active       idle   3        52.15.79.127    2379/tcp         Healthy with 3 known peers.
etcd/1*                   active       idle   4        52.15.111.66    2379/tcp         Healthy with 3 known peers. (leader)
etcd/2                    active       idle   5        52.15.144.25    2379/tcp         Healthy with 3 known peers.
kubeapi-load-balancer/0*  active       idle   7        52.15.84.179    443/tcp          Loadbalancer ready.
kubernetes-master/0*      active       idle   8        52.15.106.225   6443/tcp         Kubernetes master services ready.
flannel/3               active       idle            52.15.106.225                    Flannel subnet 10.1.48.1/24
kubernetes-worker/0*      active       idle   9        52.15.153.246                    Kubernetes worker running.
flannel/2               active       idle            52.15.153.246                    Flannel subnet 10.1.53.1/24
kubernetes-worker/1       active       idle   10       52.15.52.103                     Kubernetes worker running.
flannel/0*              active       idle            52.15.52.103                     Flannel subnet 10.1.31.1/24
kubernetes-worker/2       active       idle   11       52.15.104.181                    Kubernetes worker running.
flannel/1               active       idle            52.15.104.181                    Flannel subnet 10.1.83.1/24

Machine  State    DNS            Inst id              Series  AZ
0        started  52.15.95.92    i-06e66414008eca61c  xenial  us-east-2c
3        started  52.15.79.127   i-0038186d2c5103739  xenial  us-east-2b
4        started  52.15.111.66   i-0ac66c86a8ec93b18  xenial  us-east-2a
5        started  52.15.144.25   i-078cfe79313d598c9  xenial  us-east-2c
7        started  52.15.84.179   i-00fd70321a51b658b  xenial  us-east-2c
8        started  52.15.106.225  i-0109a5fc942c53ed7  xenial  us-east-2b
9        started  52.15.153.246  i-0ab63e34959cace8d  xenial  us-east-2b
10       started  52.15.52.103   i-0108a8cc0978954b5  xenial  us-east-2a
11       started  52.15.104.181  i-0f5562571c649f0f2  xenial  us-east-2c
```



## 与集群的交互

部署集群后，您可以在任意一个 kubernetes-master 或 kubernetes-worker 节点取得集群的控制权。

首先，您需要将凭据和客户端程序下载到本地：

创建 kubectl config 目录。

```
mkdir -p ~/.kube
```
将 kubeconfig 文件复制到默认位置。

```
juju scp kubernetes-master/0:config ~/.kube/config
```



基于已部署的系统架构，获取相应的二进制文件。如果您的客户端是另一套系统架构，您需要通过其他方式获得合适的 `kubectl` 文件。本例中，为方便起见，我们把 kubectl 复制到 `~/bin`，默认该路径在 $PATH 里。

```
mkdir -p ~/bin
juju scp kubernetes-master/0:kubectl ~/bin/kubectl
```



查询集群：

    kubectl cluster-info

输出:

```
Kubernetes master is running at https://52.15.104.227:443
Heapster is running at https://52.15.104.227:443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://52.15.104.227:443/api/v1/namespaces/kube-system/services/kube-dns/proxy
Grafana is running at https://52.15.104.227:443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
InfluxDB is running at https://52.15.104.227:443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy
```

恭喜，您已经启动了一个 Kubernetes 集群。



## 扩容集群

需要更强大的 Kubernetes 节点？通过使用 Juju 的 **约束** ，您可以轻松地请求不同规模的云资源。通过 Juju 请求创建的任意系统，您都可以为它们增加 CPU 和内存（RAM）数量。这使您可以微调 Kubernetes 集群以适应工作负载。在 bootstrap 命令上使用该标志或使用独立的 `juju constraints` 命令。请参阅 [和机器相关的 Juju 文档](https://jujucharms.com/docs/2.2/charms-constraints)



## 扩展集群

需要更多的节点？只需添加一些单元：

```shell
juju add-unit kubernetes-worker
```

或者一次添加多个：

```shell
juju add-unit -n3 kubernetes-worker
```
您也可以为特定实例类型或者特定机器的设置约束。参考 [约束文档](https://jujucharms.com/docs/stable/reference-constraints)。接下来举一些例子，请注意，诸如 `cores` 和 `mem` 这样的通用约束在各个云之间是可移植的。在本例中，我们从 AWS 申请一个特定的实例类型：

```shell
juju set-constraints kubernetes-worker instance-type=c4.large
juju add-unit kubernetes-worker
```

为提升 key/value 存储的高容错性，您也可以扩展 etcd：

```shell
juju add-unit -n3 etcd
```
强烈建议使用奇数个的法定选举单元。



## 终止集群

如果你想停止服务器，你可以销毁 Juju 模型或 controller。使用 `juju switch` 命令获取当前的 controller 名称：

```shell
juju switch
juju destroy-controller $controllername --destroy-all-models
```
这将关闭并终止该云上所有正在运行的实例。
{% endcapture %}



{% capture discussion %}
## 更多信息

Ubuntu Kubernetes 的部署通过开源软件或代码操作管理，这被称作 charms。这些 charms 以层的方式组装，从而使代码更小，更专注于 Kubernetes 及其组件的操作。

Kubernetes 的层和 Bundle 可以在 github.com 的 `kubernetes` 项目中找到：

- [Bundle 的地址](https://git.k8s.io/kubernetes/cluster/juju/bundles)
- [Kubernetes charm 层的地址](https://git.k8s.io/kubernetes/cluster/juju/layers)
- [Canonical Kubernetes home](https://jujucharms.com/kubernetes)
- [跟踪主要 issue](https://github.com/juju-solutions/bundle-canonical-kubernetes)

欢迎提供功能需求，错误报告，pull 请求和反馈意见。
{% endcapture %}

{% include templates/task.md %}
