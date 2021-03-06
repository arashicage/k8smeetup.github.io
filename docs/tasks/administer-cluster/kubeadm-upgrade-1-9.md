---
approvers:
- pipejakob
- luxas
- roberthbailey
- jbeda
title: kubeadm 集群在 1.8 和 1.9 版本之间进行升级和降级
cn-approvers:
- chentao1596
---


{% capture overview %}


本指南用于指导如何将 `kubeadm` 集群从 1.8.x 版本升级到 1.9.x 版本，也可用于从 1.8.x 升级到 1.8.y 及从 1.9.x 升级到 1.9.y，其中 `y > x`。
如果您当前使用 1.7 版本的集群，请查看 [将 kubeadm 集群从 1.7 升级到 1.8](/docs/tasks/administer-cluster/kubeadm-upgrade-1-8/)。

{% endcapture %}

{% capture prerequisites %}


开始之前：


- 您需要有一个正常工作的 1.8.0 或更高版本的 `kubeadm` Kubernetes 集群，以进行此处描述的流程。还需要禁用 swap。
- 请确保已经仔细阅读 [版本更新](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md)。
- `kubeadm upgrade` 现在允许升级您的 etcd。默认情况下，作为集群从 1.8 升级到 1.9 版本的一部分，`kubeadm upgrade` 会将 etcd 升级至 3.1.10 版本。这是因为 etcd 3.1.10 是 Kubernetes 1.9 官方验证的版本。升级由 kubeadm 自动帮您处理。
- 请注意，`kubeadm upgrade` 只会升级 Kubernetes 内建（Kubernetes-internal）组件，不会触及任何工作负载。作为最佳实践，您应该备份所有重要数据。例如，任何应用层级的状态数据，如应用可能依赖的数据库（如 MySQL 或 MongoDB）等，在开始升级前必须对其进行备份。


此外，请注意升级仅支持一个次级版本号。也就是说，您只能从 1.8 升级到 1.9 而不能从 1.7 升级到 1.9。

{% endcapture %}

{% capture steps %}


## 升级控制平面（control plane）


在 master 节点执行下面这些命令：


1. 像这样使用 `curl` 安装最新版本的 `kubeadm`：

```shell
$ export VERSION=$(curl -sSL https://dl.k8s.io/release/stable.txt) # or manually specify a released Kubernetes version
$ export ARCH=amd64 # or: arm, arm64, ppc64le, s390x
$ curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubeadm > /usr/bin/kubeadm
$ chmod a+rx /usr/bin/kubeadm
```


**警示：** 在升级控制平面前，升级系统上的 `kubeadm` 包将导致升级失败。即使 `kubeadm` 已经放入 Kubernetes 仓库中，您仍应该手动安装它。Kubeadm 团队正在修复这个限制。
{: .caution}


验证下载的 kubeadm 是否工作正常，是否为预期的版本：

```shell
$ kubeadm version
```


2. 在 master 节点上执行下列步骤：

```shell
$ kubeadm upgrade plan
[preflight] Running pre-flight checks
[upgrade] Making sure the cluster is healthy:
[upgrade/health] Checking API Server health: Healthy
[upgrade/health] Checking Node health: All Nodes are healthy
[upgrade/health] Checking Static Pod manifests exists on disk: All manifests exist on disk
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Fetching available versions to upgrade to:
[upgrade/versions] Cluster version: v1.8.1
[upgrade/versions] kubeadm version: v1.9.0
[upgrade/versions] Latest stable version: v1.9.0
[upgrade/versions] Latest version in the v1.8 series: v1.8.6

Components that must be upgraded manually after you've upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT      AVAILABLE
Kubelet     1 x v1.8.1   v1.8.6

Upgrade to the latest version in the v1.8 series:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.8.1    v1.8.6
Controller Manager   v1.8.1    v1.8.6
Scheduler            v1.8.1    v1.8.6
Kube Proxy           v1.8.1    v1.8.6
Kube DNS             1.14.4    1.14.5

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.8.6

_____________________________________________________________________

Components that must be upgraded manually after you've upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT      AVAILABLE
Kubelet     1 x v1.8.1   v1.9.0

Upgrade to the latest experimental version:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.8.1    v1.9.0
Controller Manager   v1.8.1    v1.9.0
Scheduler            v1.8.1    v1.9.0
Kube Proxy           v1.8.1    v1.9.0
Kube DNS             1.14.5    1.14.7

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.9.0

Note: Before you do can perform this upgrade, you have to update kubeadm to v1.9.0

_____________________________________________________________________
```


`kubeadm upgrade plan` 将检查您的集群是否处于可升级状态，并以用户友好的方式获取可升级的版本。


3. 选择要升级到的版本并运行。例如：

```shell
$ kubeadm upgrade apply v1.9.0
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/version] You have chosen to upgrade to version "v1.9.0"
[upgrade/versions] Cluster version: v1.8.1
[upgrade/versions] kubeadm version: v1.9.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler]
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.9.0"...
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests802453804/etcd.yaml"
[upgrade/staticpods] Moved upgraded manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests502223003/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/staticpods] Writing upgraded Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests802453804"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests802453804/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests802453804/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests802453804/kube-scheduler.yaml"
[upgrade/staticpods] Moved upgraded manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests502223003/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Moved upgraded manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests502223003/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Moved upgraded manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests502223003/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.9.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets in turn.
```


`kubeadm upgrade apply` 将执行下列步骤：

- 检查集群是否处于可升级状态，包括：
  - API Server 是否可达
  - 所有节点是否均处于 `Ready` 状态
  - 控制平面处于健康状态
- 强制启用版本偏移策略（version skew policy）。
- 保证控制平面的镜像可用或可以拉取到机器上。
- 升级控制平面组件，如果任何一个组件失败，则对升级操作进行回退。
- 应用新的 `kube-dns` 和 `kube-proxy` 清单文件并严格保证创建了所有必要的 RBAC 规则。


4. 手动升级软件定义网络（Software Defined Network，SDN）。

   当前您的容器网络接口提供商（Container Network Interface，CNI）可能有自己的升级指导。请查阅 [插件](/docs/concepts/cluster-administration/addons/) 页面，找到您的 CNI 提供商并查看是否有其他必要的升级步骤。


## 升级 master 和 node 软件包


对于集群中的每个节点（以下称为 `$HOST`），请运行下列命令升级其 `kubelet`。


1. 准备节点以进行维护，将其标记为不可调度并驱逐工作负载：

```shell
$ kubectl drain $HOST --ignore-daemonsets
```


在 master 节点上执行这个命令时，预计会出现下面这个错误，该错误是可以安全忽略的（因为 master 节点上有 static pod 运行）：

```shell
node "master" already cordoned
error: pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): etcd-kubeadm, kube-apiserver-kubeadm, kube-controller-manager-kubeadm, kube-scheduler-kubeadm
```


2. 使用 Linux 发行版特定的包管理器升级 `$HOST` 节点上的 Kubernetes 软件包版本：


如果主机正在运行基于 Debian 的发行版（如 Ubuntu），请执行： 

```shell
$ apt-get update
$ apt-get upgrade
```


如果主机正运行 CentOS 或类似发行版，请执行：

```shell
$ yum update
```


现在，节点上应该运行的是新版本的 `kubelet`。请在 `$HOST` 上执行下列命令对此进行验证：

```shell
$ systemctl status kubelet
```


3. 将节点标记为可调度（schedulable），使其重新上线：

```shell
$ kubectl uncordon $HOST
```


4. 在对集群中所有节点的 `kubelet` 进行升级之后，请执行以下命令，以确认所有节点又重新变为可用状态（从任何地方执行，例如集群外部）：

```shell
$ kubectl get nodes
```


如果上述命令结果中，所有节点的 `STATUS` 列都显示为 `Ready`，升级工作就已成功完成。


## 从故障状态恢复


如果 `kubeadm upgrade` 因某些原因失败并且不能回退（例如：执行过程中意外的关闭了节点实例），您可以再次运行 `kubeadm upgrade`，因为其具有幂等性，所以最终应该能够保证集群的实际状态就是您声明的所需状态。

 x.x.x` with `--force`, which can be used to recover from a bad state.
-->
您可以在使用 `kubeadm upgrade` 命令时带上 `x.x.x --> x.x.x` 及 `--force` 参数去变更正在运行中集群，使其从故障状态恢复。

{% endcapture %}

{% include templates/task.md %}
