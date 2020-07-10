# Kubernetes In Docker Kind 部署使用
## Kind
Kind 全称 Kubernetes In Docker，使用 golang 开发，作用为将 k8s 的组件打包到 docker 容器中，用于快速搭建 k8s 测试集群、节省资源。
注意：由于 k8s 是容器编排工具，故由于底层设计是不应该跑着容器中的，所以 Kind 不可用于实际生产环境。</br>
官网 https://kind.sigs.k8s.io/docs/user/quick-start/  </br>
https://github.com/kubernetes-sigs/kind </br>


## Kind 部署
kind linux 系统下二进制文件下载
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.7.0/kind-$(uname)-amd64
chmod +x ./kind

mkdir /sdata/usr/local/kind/bin
mv ./kind /sdata/usr/local/kind/bin
```
设置软连、查看版本
```
# ln -s /sdata/usr/local/kind/bin/kind /usr/bin/kind
# kind --version
kind version 0.7.0
```
docker 依赖
```
# docker -v
Docker version 18.09.6, build 481bc77156
```
安装 kubectl 用于管理 k8s 集群  https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl
```
yum install kubernetes-client
```
确认 kubectl 版本
```
# kubectl version --client
Client Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.2", GitCommit:"269f928217957e7126dc87e6adfa82242bfe5b1e", GitTreeState:"clean", BuildDate:"2017-07-03T15:31:10Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"linux/amd64"}
```
yum 直接安装的 kubectl 版本较低需升级，此处变更 yum 的镜像源为阿里云 repo
```
# 卸载旧版本
yum remove kubernetes-client

# 设置镜像源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 重新安装
yum install -y kubectl

# 版本号
kubectl version --client
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.2", GitCommit:"52c56ce7a8272c798dbc29846288d7cd9fbae032", GitTreeState:"clean", BuildDate:"2020-04-16T11:56:40Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

## Kind 创建 k8s
### k8s 单节点创建
创建单节点 k8s，--name 参数命名
```
kind create cluster --name mycluster
```
查看集群信息
```
# kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:32769
KubeDNS is running at https://127.0.0.1:32769/api/v1/proxy/namespaces/kube-system/services/kube-dns
```
查看集群节点信息
```
# kubectl get nodes
NAME                      STATUS    AGE
mycluster-control-plane   Ready     9m
```
查看 kind 建设的 k8s 集群信息
```
# kind get clusters
mycluster
```
删除集群
```
# kind delete cluster --name mycluster
Deleting cluster "mycluster" ...

# kubectl cluster-info
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
### k8s 多节点集群创建
Kind 需使用配置文件来定义 k8s 多节点集群的创建，需创建 yaml 文件来定义参数。</br>
创建 1 个 control-plane nodes 和 1 个 workers，需注意网络规划。</br>
https://kind.sigs.k8s.io/docs/user/configuration/</br>
```
vim kind-k8s-config.yaml

# a cluster with 3 control-plane nodes and 3 workers
kind: Cluster
# kind api version
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```
根据配置创建集群
```
# kind create cluster --config kind-k8s-config.yaml
```
## k8s 使用
### 基本信息
查看 k8s 节点
```
# kubectl get nodes

NAME                 STATUS   ROLES    AGE    VERSION
kind-control-plane   Ready    master   105m   v1.17.0
kind-worker          Ready    <none>   104m   v1.17.0
kind-worker2         Ready    <none>   104m   v1.17.0
```
查看节点详细信息，其中，kube-system 是 Kubernetes 项目预留的系统 Pod 的工作空间（Namepsace，注意它并不是 Linux Namespace，它只是 Kubernetes 划分不同工作空间的单位）。
```
# kubectl describe node kind-control-plane
Name:               kind-control-plane
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kind-control-plane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    scheduler.alpha.kubernetes.io/taints: []
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 30 Apr 2020 15:06:16 +0800
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  kind-control-plane
  AcquireTime:     <unset>
  RenewTime:       Thu, 30 Apr 2020 16:53:15 +0800
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Thu, 30 Apr 2020 16:48:29 +0800   Thu, 30 Apr 2020 15:06:15 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Thu, 30 Apr 2020 16:48:29 +0800   Thu, 30 Apr 2020 15:06:15 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Thu, 30 Apr 2020 16:48:29 +0800   Thu, 30 Apr 2020 15:06:15 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Thu, 30 Apr 2020 16:48:29 +0800   Thu, 30 Apr 2020 15:08:21 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.17.0.4
  Hostname:    kind-control-plane
Capacity:
  cpu:                4
  ephemeral-storage:  206291944Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8008816Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  206291944Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8008816Ki
  pods:               110
System Info:
  Machine ID:                 dc3dc748697c4681b1f9071d8e33832f
  System UUID:                0be88559-9de8-4cc0-aa83-07ced946aeb1
  Boot ID:                    35cd74bb-d0e8-4570-a801-0d6a1a810863
  Kernel Version:             3.10.0-1062.18.1.el7.x86_64
  OS Image:                   Ubuntu 19.10
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.3.2
  Kubelet Version:            v1.17.0
  Kube-Proxy Version:         v1.17.0
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (9 in total)
  Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                          ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-6955765f44-frrnk                      100m (2%)     0 (0%)      70Mi (0%)        170Mi (2%)     106m
  kube-system                 coredns-6955765f44-smc57                      100m (2%)     0 (0%)      70Mi (0%)        170Mi (2%)     106m
  kube-system                 etcd-kind-control-plane                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         106m
  kube-system                 kindnet-n58p5                                 100m (2%)     100m (2%)   50Mi (0%)        50Mi (0%)      106m
  kube-system                 kube-apiserver-kind-control-plane             250m (6%)     0 (0%)      0 (0%)           0 (0%)         106m
  kube-system                 kube-controller-manager-kind-control-plane    200m (5%)     0 (0%)      0 (0%)           0 (0%)         106m
  kube-system                 kube-proxy-lpkwr                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         106m
  kube-system                 kube-scheduler-kind-control-plane             100m (2%)     0 (0%)      0 (0%)           0 (0%)         106m
  local-path-storage          local-path-provisioner-7745554f7f-jl7bb       0 (0%)        0 (0%)      0 (0%)           0 (0%)         106m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                850m (21%)  100m (2%)
  memory             190Mi (2%)  390Mi (4%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>




# kubectl describe node kind-worker2
Name:               kind-worker2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kind-worker2
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 30 Apr 2020 15:07:41 +0800
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  kind-worker2
  AcquireTime:     <unset>
  RenewTime:       Thu, 30 Apr 2020 16:53:44 +0800
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Thu, 30 Apr 2020 16:50:16 +0800   Thu, 30 Apr 2020 15:07:41 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Thu, 30 Apr 2020 16:50:16 +0800   Thu, 30 Apr 2020 15:07:41 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Thu, 30 Apr 2020 16:50:16 +0800   Thu, 30 Apr 2020 15:07:41 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Thu, 30 Apr 2020 16:50:16 +0800   Thu, 30 Apr 2020 15:10:13 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.17.0.5
  Hostname:    kind-worker2
Capacity:
  cpu:                4
  ephemeral-storage:  206291944Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8008816Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  206291944Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8008816Ki
  pods:               110
System Info:
  Machine ID:                 2fbdb65ce42648039a7096443177bd28
  System UUID:                5a2a0bac-4c83-4bce-a268-69fdf32c7f6a
  Boot ID:                    35cd74bb-d0e8-4570-a801-0d6a1a810863
  Kernel Version:             3.10.0-1062.18.1.el7.x86_64
  OS Image:                   Ubuntu 19.10
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.3.2
  Kubelet Version:            v1.17.0
  Kube-Proxy Version:         v1.17.0
PodCIDR:                      10.244.2.0/24
PodCIDRs:                     10.244.2.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                ------------  ----------  ---------------  -------------  ---
  kube-system                 kindnet-528sk       100m (2%)     100m (2%)   50Mi (0%)        50Mi (0%)      105m
  kube-system                 kube-proxy-qhtpg    0 (0%)        0 (0%)      0 (0%)           0 (0%)         105m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (2%)  100m (2%)
  memory             50Mi (0%)  50Mi (0%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:              <none>
```
查看 Namespace 中 pod 状态</br>
网络相关 coredns、kube-controller-manager，master 相关 kube-apiserver、kube-controller-manager、kube-scheduler
```
# kubectl get pods -n kube-system
NAME                                         READY     STATUS    RESTARTS   AGE
coredns-6955765f44-frrnk                     1/1       Running   0          16m
coredns-6955765f44-smc57                     1/1       Running   0          16m
etcd-kind-control-plane                      1/1       Running   0          17m
kindnet-528sk                                1/1       Running   0          15m
kindnet-dwzhs                                1/1       Running   0          16m
kindnet-n58p5                                1/1       Running   0          16m
kube-apiserver-kind-control-plane            1/1       Running   1          17m
kube-controller-manager-kind-control-plane   1/1       Running   3          17m
kube-proxy-lpkwr                             1/1       Running   0          16m
kube-proxy-qhtpg                             1/1       Running   0          15m
kube-proxy-rqm2h                             1/1       Running   0          16m
kube-scheduler-kind-control-plane            1/1       Running   3          17m
```

