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
## kubeadm
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
