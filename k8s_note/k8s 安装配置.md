# 资源依赖
## 硬件资源
master 至少 2C 4GB，推荐 4C 16GB</br>
node 至少 4C 16GB，具体根据 pod 数量界定

## 软件依赖
etcd > version 3.0</br>
docker > version 18.03，推荐 Docker CE 18.09</br>

## 系统配置
完成以下修改后需重启服务器
### 禁用交换内存
kubeadm 推荐关闭交换空间使用，提高性能</br>
```
[root@vmw-dev-k8s-01 ~]# swapoff -a

[root@vmw-dev-k8s-01 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 禁用防火墙
在安全网络内关闭防火墙
```
[root@vmw-dev-k8s-01 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 三 2020-01-29 18:08:25 CST; 40min ago
     Docs: man:firewalld(1)
 Main PID: 5579 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─5579 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

1月 29 18:08:19 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
1月 29 18:08:25 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.

[root@vmw-dev-k8s-01 ~]# systemctl stop firewalld
[root@vmw-dev-k8s-01 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

### 关闭 selinux
```
[root@vmw-dev-k8s-01 ~]# setenforce 0
```

# k8s 安装
## kubeadm 安装
最好不要使用 yum install kubernetes 来安装 k8s 集群，因为使用此命令安装需要额外的配置。</br>
k8s 官方提供了 kubeadm 工具简化 k8s 安装，k8s官网 yum 源是packages.cloud.google.com。国内无法访问，需修改为阿里云 yum 源。
```
[root@vmw-dev-k8s-01 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=0
> EOF
```
yum 安装 kubelet kubeadm kubectl
```
[root@vmw-dev-k8s-01 ~]# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
yum 安装 docker 
```
yum install -y docker
```
设置开机启动
```
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```
此处不能直接使用 systemctl start kubelet 命令启动 kubelet，否则会报错
```
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since 四 2020-01-30 21:18:43 CST; 2s ago
     Docs: https://kubernetes.io/docs/
  Process: 7126 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 7126 (code=exited, status=255)

1月 30 21:18:43 vmw-dev-k8s-01 systemd[1]: kubelet.service: main process exited, code=exited, st...n/a
1月 30 21:18:43 vmw-dev-k8s-01 systemd[1]: Unit kubelet.service entered failed state.
1月 30 21:18:43 vmw-dev-k8s-01 systemd[1]: kubelet.service failed.
Hint: Some lines were ellipsized, use -l to show in full.

# journalctl -f -u kubelet
-- Logs begin at 四 2020-01-30 20:40:49 CST. --
1月 30 21:17:00 vmw-dev-k8s-01 systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
1月 30 21:17:00 vmw-dev-k8s-01 systemd[1]: Unit kubelet.service entered failed state.
1月 30 21:17:00 vmw-dev-k8s-01 systemd[1]: kubelet.service failed.
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: kubelet.service holdoff time over, scheduling restart.
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
1月 30 21:17:10 vmw-dev-k8s-01 kubelet[7079]: F0130 21:17:10.462216    7079 server.go:198] failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: Unit kubelet.service entered failed state.
1月 30 21:17:10 vmw-dev-k8s-01 systemd[1]: kubelet.service failed.
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: kubelet.service holdoff time over, scheduling restart.
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
1月 30 21:17:20 vmw-dev-k8s-01 kubelet[7085]: F0130 21:17:20.875065    7085 server.go:198] failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: Unit kubelet.service entered failed state.
1月 30 21:17:20 vmw-dev-k8s-01 systemd[1]: kubelet.service failed.
```
提示缺失 /var/lib/kubelet/config.yaml，需要先执行 kubeadm init
```
# kubeadm init
W0130 21:27:03.717998    8070 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0130 21:27:03.718727    8070 version.go:102] falling back to the local client version: v1.17.2
W0130 21:27:03.719755    8070 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0130 21:27:03.719797    8070 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.2
[preflight] Running pre-flight checks
	[WARNING Hostname]: hostname "vmw-dev-k8s-01" could not be reached
	[WARNING Hostname]: hostname "vmw-dev-k8s-01": lookup vmw-dev-k8s-01 on 192.168.45.2:53: no such host
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
提示 CPU 核数至少为 2 ，此处为虚拟测试环境直接忽略
```
# kubeadm init --ignore-preflight-errors=NumCPU
W0130 21:33:28.243307    8315 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0130 21:33:28.245062    8315 version.go:102] falling back to the local client version: v1.17.2
W0130 21:33:28.246511    8315 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0130 21:33:28.246568    8315 validation.go:28] Cannot validate kubelet config - no validator is available
[init] Using Kubernetes version: v1.17.2
[preflight] Running pre-flight checks
	[WARNING NumCPU]: the number of available CPUs 1 is less than the required 2
	[WARNING Hostname]: hostname "vmw-dev-k8s-01" could not be reached
	[WARNING Hostname]: hostname "vmw-dev-k8s-01": lookup vmw-dev-k8s-01 on 192.168.45.2:53: no such host
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
提示  /proc/sys/net/bridge/bridge-nf-call-iptables 必须设置为 1
```
echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables
```
设置后重试，提示拉取镜像超时
```
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.17.2: output: Trying to pull repository k8s.gcr.io/kube-apiserver ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 74.125.203.82:443: connect: connection refused
, error: exit status 1
```
查看需要安装镜像列表
```
# kubeadm config images list
W0130 21:56:47.040526    9267 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
W0130 21:56:47.040865    9267 version.go:102] falling back to the local client version: v1.17.2
W0130 21:56:47.041245    9267 validation.go:28] Cannot validate kubelet config - no validator is available
W0130 21:56:47.041280    9267 validation.go:28] Cannot validate kube-proxy config - no validator is available
k8s.gcr.io/kube-apiserver:v1.17.2
k8s.gcr.io/kube-controller-manager:v1.17.2
k8s.gcr.io/kube-scheduler:v1.17.2
k8s.gcr.io/kube-proxy:v1.17.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.5
```
输出初始化参数文件
```
# kubeadm config print init-defaults > init.default.yaml
W0130 22:15:23.565860    9896 validation.go:28] Cannot validate kubelet config - no validator is available
W0130 22:15:23.566048    9896 validation.go:28] Cannot validate kube-proxy config - no validator is available

# cat init.default.yaml 
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: vmw-dev-k8s-01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.17.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```
对 init.default.yaml 进行编辑，对 k8s 默认仓库进行修改，保存为 init-config.yaml 
```
[root@vmw-dev-k8s-01 kubernetes]# vim init-config.yaml 

apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.45.128  # master 节点 ip
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: 192.168.45.128  # master 节点 ip，若填主机名需保证解析
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd   # 把etcd容器的目录挂载到本地的/var/lib/etcd目录下，防止数据丢失
imageRepository: docker.io/dustise   # 镜像仓库地址
kind: ClusterConfiguration
kubernetesVersion: v1.17.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}

```
设置 docker 国内加速器
```
# echo '{"data-root":"/sdata/docker","registry-mirros":["https://registry.docker-cn.com"]}' > /etc/docker/deamon.json
```
重启 docker 
```
systemctl restart docker
```
使用 k8s 新配置文件拉取所需镜像
```
[root@vmw-dev-k8s-01 kubernetes]# kubeadm config images pull --config=init-config.yaml
W0130 22:43:29.713082   10935 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0130 22:43:29.713214   10935 validation.go:28] Cannot validate kubelet config - no validator is available
failed to pull image "docker.io/dustise/kube-apiserver:v1.17.0": output: Trying to pull repository docker.io/dustise/kube-apiserver ... 
manifest for docker.io/dustise/kube-apiserver:v1.17.0 not found
, error: exit status 1
To see the stack trace of this error execute with --v=5 or higher
```
提示没有 1.17.0 这个版本的镜像，但由于当前版本 kubeadm 不能使用低于 1.16.0 版本的 k8s，故修改 gcr.azk8s.cn/google_containers
```
imageRepository:  gcr.azk8s.cn/google_containers   # 镜像仓库地址
kind: ClusterConfiguration
kubernetesVersion: v1.17.0
```
根据新配置文件拉取镜像
```
# kubeadm config images pull --config=init-config.yaml
W0130 22:49:24.763926   11131 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0130 22:49:24.764040   11131 validation.go:28] Cannot validate kubelet config - no validator is available
[config/images] Pulled gcr.azk8s.cn/google_containers/kube-apiserver:v1.17.0
[config/images] Pulled gcr.azk8s.cn/google_containers/kube-controller-manager:v1.17.0
[config/images] Pulled gcr.azk8s.cn/google_containers/kube-scheduler:v1.17.0
[config/images] Pulled gcr.azk8s.cn/google_containers/kube-proxy:v1.17.0
[config/images] Pulled gcr.azk8s.cn/google_containers/pause:3.1
[config/images] Pulled gcr.azk8s.cn/google_containers/etcd:3.4.3-0
[config/images] Pulled gcr.azk8s.cn/google_containers/coredns:1.6.5
```
查看 docker 镜像
```
# docker images
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
gcr.azk8s.cn/google_containers/kube-proxy                v1.17.0             7d54289267dc        7 weeks ago         116 MB
gcr.azk8s.cn/google_containers/kube-apiserver            v1.17.0             0cae8d5cc64c        7 weeks ago         171 MB
gcr.azk8s.cn/google_containers/kube-controller-manager   v1.17.0             5eb3b7486872        7 weeks ago         161 MB
gcr.azk8s.cn/google_containers/kube-scheduler            v1.17.0             78c190f736b1        7 weeks ago         94.4 MB
gcr.azk8s.cn/google_containers/coredns                   1.6.5               70f311871ae1        2 months ago        41.6 MB
gcr.azk8s.cn/google_containers/etcd                      3.4.3-0             303ce5db0e90        3 months ago        288 MB
gcr.azk8s.cn/google_containers/pause                     3.1                 da86e6ba6ca1        2 years ago         742 kB
```
根据配置文件，初始化安装 k8s master
```
# kubeadm init --config=init-config.yaml  --ignore-preflight-errors=NumCPU
[kubelet-check] Initial timeout of 40s passed.

Unfortunately, an error has occurred:
	timed out waiting for the condition

This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
	- 'systemctl status kubelet'
	- 'journalctl -xeu kubelet'

Additionally, a control plane component may have crashed or exited when started by the container runtime.
```
提示 kubelket 服务未启动，拉完镜像后需要先启动 kubelet 再进行 init
```
# systemctl start kubelet
# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since 四 2020-01-30 23:11:13 CST; 10s ago
     Docs: https://kubernetes.io/docs/
 Main PID: 16230 (kubelet)
   CGroup: /system.slice/kubelet.service
           └─16230 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --...

1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:23.617240   16230 kubelet.go:2263] no...und
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:23.718456   16230 kubelet.go:2263] no...und
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:23.760041   16230 reflector.go:153] k...sed
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: I0130 23:11:23.763200   16230 kubelet_node_status...ach
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: I0130 23:11:23.773849   16230 kubelet_node_status...128
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:23.819246   16230 kubelet.go:2263] no...und
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:23.919845   16230 kubelet.go:2263] no...und
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: W0130 23:11:23.958008   16230 status_manager.go:530]...
1月 30 23:11:23 vmw-dev-k8s-01 kubelet[16230]: W0130 23:11:23.992313   16230 cni.go:237] Unable ...t.d
1月 30 23:11:24 vmw-dev-k8s-01 kubelet[16230]: E0130 23:11:24.020547   16230 kubelet.go:2263] no...und
Hint: Some lines were ellipsized, use -l to show in full.
```
重新 init 会报错，显示部分 docker images 及配置文件已存在
```
# docker ps -a | grep kube | grep -v pause
f8314330d651        0cae8d5cc64c                               "kube-apiserver --..."   20 seconds ago      Exited (1) 17 seconds ago                       k8s_kube-apiserver_kube-apiserver-192.168.45.128_kube-system_398df73673d361199c6c20a9c254e739_30
3fdddf4b34e8        5eb3b7486872                               "kube-controller-m..."   21 seconds ago      Exited (1) 15 seconds ago                       k8s_kube-controller-manager_kube-controller-manager-192.168.45.128_kube-system_ba8dda8b1a8bb912a9a71bbb8f225d3d_29
6bc0ca09a6dd        303ce5db0e90                               "etcd --advertise-..."   22 seconds ago      Exited (1) 21 seconds ago                       k8s_etcd_etcd-192.168.45.128_kube-system_6c39f6c38e2e58352df6adbe8d05f843_35
4e7814b1248a        78c190f736b1                               "kube-scheduler --..."   12 minutes ago      Up 12 minutes                                   k8s_kube-scheduler_kube-scheduler-192.168.45.128_kube-system_8470cb06b5da19ba2e447333859ecefa_0
```
删除已存在的配置文件
```
# cd /etc/kubernetes/manifests/
# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
# rm -rf ./*.yaml
```
停止并删除所有已在运行容器
```
# docker stop $(docker ps -q)
4e7814b1248a
1b43560478da
42e8aae812b0
84c77ee9f128
23ff50db42d1
# docker rm $(docker ps -a -q)
f0035f8eb544
3b83fb7141bd
64e1e75e249e
4e7814b1248a
1b43560478da
42e8aae812b0
84c77ee9f128
23ff50db42d1
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
重新 init 依然报端口占用，则需重启 kubeadm </br>
注意：kubeadm reset 后会自动删除 init 所创建的容器并重启 kubeadmin 和 kubelet
```
# kubeadm reset
```
重新 init 后出现以下报错
```
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: I0130 23:32:18.396612   24790 kubelet_node_status.go:294] Setting node annotation to enable volume controller attach/detach
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.398863   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: I0130 23:32:18.401270   24790 kubelet_node_status.go:70] Attempting to register node 192.168.45.128
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.499656   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.595154   24790 reflector.go:153] k8s.io/kubernetes/pkg/kubelet/kubelet.go:458: Failed to list *v1.Node: Get https://192
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.600534   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.701027   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.802022   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.902912   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:18 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:18.994496   24790 reflector.go:153] k8s.io/kubernetes/pkg/kubelet/kubelet.go:449: Failed to list *v1.Service: Get https://
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.003784   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.104827   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.194404   24790 kubelet_node_status.go:92] Unable to register node "192.168.45.128" with API server: Post https://192.16
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.205778   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: W0130 23:32:19.251770   24790 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.306694   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.394130   24790 reflector.go:153] k8s.io/kubernetes/pkg/kubelet/config/apiserver.go:46: Failed to list *v1.Pod: Get http
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.407617   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.508574   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.594976   24790 reflector.go:153] k8s.io/client-go/informers/factory.go:135: Failed to list *v1beta1.CSIDriver: Get http
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.609548   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.710498   24790 kubelet.go:2263] node "192.168.45.128" not found
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: E0130 23:32:19.795086   24790 csi_plugin.go:267] Failed to initialize CSINodeInfo: error updating CSINode annotation: timed out waitin
1月 30 23:32:19 vmw-dev-k8s-01 kubelet[24790]: F0130 23:32:19.795126   24790 csi_plugin.go:281] Failed to initialize CSINodeInfo after retrying
1月 30 23:32:19 vmw-dev-k8s-01 systemd[1]: kubelet.service: main process exited, code=exited, status=255/n/a
1月 30 23:32:19 vmw-dev-k8s-01 systemd[1]: Unit kubelet.service entered failed state.
1月 30 23:32:19 vmw-dev-k8s-01 systemd[1]: kubelet.service failed.

```
发现 yum 安装 docker 版本未达到要求，卸载后更新 docker 版本</br>
注意：更新好重新拉镜像 kubeadm config images pull --config=init-config.yaml
```
# docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:25:41 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea
  Built:            Wed Nov 13 07:24:18 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

```
重新 kubeadm init 后出现新报错
```
1月 31 01:00:09 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:09.263098   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:09 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:09.519567   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:11 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:11.654901   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:11 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:11.810774   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.023497   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.178529   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.207479   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.272782   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.366089   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.440707   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:12 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:12.893375   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:13 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:13.053711   57148 event.go:263] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta
1月 31 01:00:16 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:16.812536   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:17 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:17.573629   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:21 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:21.814068   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:22 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:22.598542   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:26 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:26.814694   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:28 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:28.014471   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:31 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:31.826612   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:35 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:35.033249   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:36 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:36.872487   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:40 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:40.112232   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:42 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:42.546173   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:47 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:47.034225   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m
1月 31 01:00:47 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:47.546464   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:53 vmw-dev-k8s-01 kubelet[57148]: W0131 01:00:52.992057   57148 cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
1月 31 01:00:54 vmw-dev-k8s-01 kubelet[57148]: E0131 01:00:54.069958   57148 kubelet.go:2183] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady m

```
```
# kubectl get nodes
NAME             STATUS     ROLES    AGE     VERSION
vmw-dev-k8s-01   NotReady   <none>   9m28s   v1.17.2

# kubectl get pods -n kube-system -o wide |grep vmw-dev-k8s-01
etcd-vmw-dev-k8s-01                      1/1     Running   0          9m27s   192.168.45.128   vmw-dev-k8s-01   <none>           <none>
kube-apiserver-vmw-dev-k8s-01            1/1     Running   2          11m     192.168.45.128   vmw-dev-k8s-01   <none>           <none>
kube-controller-manager-vmw-dev-k8s-01   1/1     Running   2          11m     192.168.45.128   vmw-dev-k8s-01   <none>           <none>
kube-scheduler-vmw-dev-k8s-01            1/1     Running   2          11m     192.168.45.128   vmw-dev-k8s-01   <none>           <none>

# kubectl --namespace kube-system logs kube-apiserver-vmw-dev-k8s-01 

[root@vmw-dev-k8s-01 kubernetes]# kubectl describe pod kube-apiserver-vmw-dev-k8s-01 --namespace=kube-system
Name:                 kube-apiserver-vmw-dev-k8s-01
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 vmw-dev-k8s-01/192.168.45.128
Start Time:           Fri, 31 Jan 2020 00:55:23 +0800
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 398df73673d361199c6c20a9c254e739
                      kubernetes.io/config.mirror: 398df73673d361199c6c20a9c254e739
                      kubernetes.io/config.seen: 2020-01-31T00:55:17.110769358+08:00
                      kubernetes.io/config.source: file
Status:               Running
IP:                   192.168.45.128
IPs:
  IP:           192.168.45.128
Controlled By:  Node/vmw-dev-k8s-01
Containers:
  kube-apiserver:
    Container ID:  docker://935dc6609b5a7b89fe44b819881a780066d5a47573c21e1f7fe6ccdbddada9df
    Image:         gcr.azk8s.cn/google_containers/kube-apiserver:v1.17.0
    Image ID:      docker-pullable://gcr.azk8s.cn/google_containers/kube-apiserver@sha256:e3ec33d533257902ad9ebe3d399c17710e62009201a7202aec941e351545d662
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=192.168.45.128
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --insecure-port=0
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Fri, 31 Jan 2020 00:59:21 +0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 31 Jan 2020 00:57:38 +0800
      Finished:     Fri, 31 Jan 2020 00:59:17 +0800
    Ready:          True
    Restart Count:  2
    Requests:
      cpu:        250m
    Liveness:     http-get https://192.168.45.128:6443/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/pki from etc-pki (ro)
      /etc/ssl/certs from ca-certs (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-pki:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/pki
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:
  Type     Reason     Age                From                     Message
  ----     ------     ----               ----                     -------
  Warning  Unhealthy  16m                kubelet, vmw-dev-k8s-01  Liveness probe failed: Get https://192.168.45.128:6443/healthz: net/http: TLS handshake timeout
  Normal   Created    15m (x2 over 17m)  kubelet, vmw-dev-k8s-01  Created container kube-apiserver
  Normal   Started    15m (x2 over 17m)  kubelet, vmw-dev-k8s-01  Started container kube-apiserver
  Warning  Unhealthy  14m                kubelet, vmw-dev-k8s-01  Liveness probe failed: Get https://192.168.45.128:6443/healthz: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  14m (x8 over 16m)  kubelet, vmw-dev-k8s-01  Liveness probe failed: HTTP probe failed with statuscode: 403
  Warning  Unhealthy  13m (x6 over 14m)  kubelet, vmw-dev-k8s-01  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    13m (x2 over 15m)  kubelet, vmw-dev-k8s-01  Container kube-apiserver failed liveness probe, will be restarted
  Normal   Pulled     13m (x3 over 17m)  kubelet, vmw-dev-k8s-01  Container image "gcr.azk8s.cn/google_containers/kube-apiserver:v1.17.0" already present on machine

```
