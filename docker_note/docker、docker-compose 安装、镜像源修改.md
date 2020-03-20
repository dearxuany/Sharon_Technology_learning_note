# docker、docker-compose 安装、镜像源修改
## docker
安装依赖
```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```  
获取 docker-ce 稳定版本 yum 镜像
```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
安装 docker 社区版
注：需注意版本基础环境内核会影响 docker 
```
yum update
yum install docker-ce-18.09.6 docker-ce-cli-18.09.6 containerd.io
```
解锁添加 docker 组
```
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
groupadd docker
```
启动 docker 设置开机自启
```
systemctl start docker
systemctl enable docker
```
查看 docker 版本
```
# docker version
Client:
Version:           18.09.6
API version:       1.39
Go version:        go1.10.8
Git commit:        481bc77156
Built:             Sat May  4 02:34:58 2019
OS/Arch:           linux/amd64
Experimental:      false

Server: Docker Engine - Community
Engine:
  Version:          18.09.6
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       481bc77
  Built:            Sat May  4 02:02:43 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```
修改 docker 镜像源
```
vim /etc/docker/daemon.json

{

"data-root":"/sdata/docker",
"registry-mirrors": ["http://hub-mirror.c.163.com"],
"insecure-registries": ["10.0.0.3:5000"]
}
```
重启 docker
```
systemctl restart docker
```
## 安装 docker-compose
注：使用 yum 安装版本较低，建议使用 pip 安装
```
yum -y install python-pip
pip install --upgrade pip
pip --default-timeout=200 install -U docker-compose
```
查看 docker-compose 版本
```
# docker-compose --version
docker-compose version 1.25.4, build unknown
```
