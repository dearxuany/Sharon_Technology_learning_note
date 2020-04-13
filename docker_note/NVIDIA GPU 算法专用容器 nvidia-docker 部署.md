# NVIDIA GPU 算法专用容器 nvidia-docker 部署
原生 docker 不支持直接使用 Nvidia GPU 作为硬件资源，nvidia-docker 是 nvidia 对原生 docker 的封装处理以支持使用  Nvidia GPU 作为容器资源，官网地址  https://github.com/NVIDIA/nvidia-docker </br>

![](https://cloud.githubusercontent.com/assets/3028125/12213714/5b208976-b632-11e5-8406-38d379ec46aa.png)



## NVIDIA driver
### 查看版本信息
nvidia 官方文档  https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#redhat-installation </br>
注意：需注意系统、内核、GCC版本等底层参数</br>
</br>
查看 CUDA-capable GPU 信息
```
# lspci | grep -i nvidia
00:08.0 3D controller: NVIDIA Corporation GP104GL [Tesla P4] (rev a1)
```
查看系统版本信息
```
# uname -m && cat /etc/*release
x86_64
CentOS Linux release 7.7.1908 (Core)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"


CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"


CentOS Linux release 7.7.1908 (Core)
CentOS Linux release 7.7.1908 (Core)
```
安装 gcc 
```
# yum -y install gcc
# gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39)
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
查看内核版本
```
# uname -r
3.10.0-1062.1.2.el7.x86_64
```
安装  kernel headers and development packages
```
# yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Package kernel-devel-3.10.0-1062.1.2.el7.x86_64 already installed and latest version
Package kernel-headers-3.10.0-1062.1.2.el7.x86_64 already installed and latest version
```
### RPM 安装  NVIDIA driver
rpm 安装依赖
```
# yum install libvdpau dkms epel-release
```
根据系统信息官网查询 cuda 版本
 https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=deblocal
```
linux > x86_64 > centos > version 7 > rpm(local)
```
下载镜像
```
# wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-rhel7-10-2-local-10.2.89-440.33.01-1.0-1.x86_64.rpm
# rpm -i cuda-repo-rhel7-10-2-local-10.2.89-440.33.01-1.0-1.x86_64.rpm
# yum clean all
# yum -y install nvidia-driver-latest-dkms cuda
# yum -y install cuda-drivers
```
添加环境变量
```
export PATH=/usr/local/cuda-10.2/bin:/usr/local/cuda-10.2/NsightCompute-2019.1${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-10.2/lib64\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
重新加载
```
source ~/.bashrc
```
查看 CUDA 版本
```
# nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Wed_Oct_23_19:24:38_PDT_2019
Cuda compilation tools, release 10.2, V10.2.89
```
验证 CUDA Driver
```
# cd /usr/local/cuda-10.2/samples/1_Utilities/deviceQuery
# make
# ./deviceQuery
./deviceQuery Starting...


CUDA Device Query (Runtime API) version (CUDART static linking)


Detected 1 CUDA Capable device(s)


Device 0: "Tesla P4"
  CUDA Driver Version / Runtime Version          10.2 / 10.2
  CUDA Capability Major/Minor version number:    6.1
  Total amount of global memory:                 7612 MBytes (7981694976 bytes)
  (20) Multiprocessors, (128) CUDA Cores/MP:     2560 CUDA Cores
  GPU Max Clock rate:                            1114 MHz (1.11 GHz)
  Memory Clock rate:                             3003 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     No
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Enabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 0 / 8
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >


deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.2, CUDA Runtime Version = 10.2, NumDevs = 1
Result = PASS

```

## nvidia-docker
### docker-ce 
官网地址 https://docs.docker.com/engine/install/centos/  </br>
安装依赖
```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
获取 docker-ce  稳定版本 yum 镜像
```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
查看发行版本
```
# yum list docker-ce --showduplicates | sort -r
```
安装 docker-ce 19.03 
```
# yum install docker-ce-19.03.5 docker-ce-cli-19.03.5 containerd.io
```
新增 docker 用户组
```
# chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
# groupadd docker
# chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow
```
修改 docker 存储目录、镜像源等
```
# vim /etc/docker/daemon.json
{
"data-root":"/sdata/docker",
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
启动 docker 设置开机自启
```
# systemctl start docker
# systemctl enable docker
```
查看 docker 版本
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
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```
### NVIDIA Container Toolkit
下载镜像
```
# distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
# # curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
[libnvidia-container]
name=libnvidia-container
baseurl=https://nvidia.github.io/libnvidia-container/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/libnvidia-container/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[nvidia-container-runtime]
name=nvidia-container-runtime
baseurl=https://nvidia.github.io/nvidia-container-runtime/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/nvidia-container-runtime/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt


[nvidia-docker]
name=nvidia-docker
baseurl=https://nvidia.github.io/nvidia-docker/centos7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://nvidia.github.io/nvidia-docker/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
安装并重启 docker 
```
# yum install -y nvidia-container-toolkit
# systemctl restart docker
```
## nvidia-docker 使用测试
使用单个 GPU 启动容器
```
#docker run --gpus 1 nvidia/cuda:10.0-base nvidia-smi

Unable to find image 'nvidia/cuda:10.0-base' locally
10.0-base: Pulling from nvidia/cuda
7ddbc47eeb70: Pull complete
c1bbdc448b72: Pull complete
8c3b70e39044: Pull complete
45d437916d57: Pull complete
d8f1569ddae6: Pull complete
de5a2c57c41d: Pull complete
ea6f04a00543: Pull complete
Digest: sha256:e6e1001f286d084f8a3aea991afbcfe92cd389ad1f4883491d43631f152f175e
Status: Downloaded newer image for nvidia/cuda:10.0-base
Mon Apr 13 01:30:29 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.33.01    Driver Version: 440.33.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla P4            On   | 00000000:00:08.0 Off |                    0 |
| N/A   30C    P8     7W /  75W |      0MiB /  7611MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
其他 GPU 调用参数启动方式
```
#### Test nvidia-smi with the latest official CUDA image
docker run --gpus all nvidia/cuda:10.0-base nvidia-smi

# Start a GPU enabled container on two GPUs
docker run --gpus 2 nvidia/cuda:10.0-base nvidia-smi

# Starting a GPU enabled container on specific GPUs
docker run --gpus '"device=1,2"' nvidia/cuda:10.0-base nvidia-smi
docker run --gpus '"device=UUID-ABCDEF,1"' nvidia/cuda:10.0-base nvidia-smi

# Specifying a capability (graphics, compute, ...) for my container
# Note this is rarely if ever used this way
docker run --gpus all,capabilities=utility nvidia/cuda:10.0-base nvidia-smi
```
