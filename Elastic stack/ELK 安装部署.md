# ELK 安装部署
## 安装java
Elasticsearch requires at least Java 8. </br>
https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html </br>
https://www.java.com/zh_CN/download/manual.jsp </br>
https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/ </br>
</br>
系统信息</br>
```
$ uname -r
3.10.0-862.2.3.el7.centos.plus.i686
```
配置YUM源
```
$ cd /etc/yum.repos.d
$ sudo vi centos.repo
```
加入以下信息
```
/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#released updates 
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=<a href="../../../../../etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7"><code>file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7</code></a>

```
安装jdk，一般推荐使用 Oracle JDK 1.8 或者 OpenJDK 1.8。我们这里使用 OpenJDK 1.8。
```
sudo yum install java-1.8.0-openjdk
```
验证java
```
$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-b12)
OpenJDK Server VM (build 25.191-b12, mixed mode)
```
## 安装 Elasticsearch
https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_installation.html#

下载  Elasticsearch 5.6.16 压缩包
```
sudo curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.16.tar.gz
```
解压安装包
```
sudo tar -xvf elasticsearch-5.6.16.tar.gz
```
配置ES
https://my.oschina.net/itblog/blog/547250/
```
cd /usr/local/elasticsearch-5.6.16/config
sudo vim elasticsearch.yml
```
加入以下一句
```
network.host: localhost
```
进入目录启动ES
```
cd elasticsearch-5.6.16/bin
./elasticsearch
```
出现错误，权限不够
```
Exception in thread "main" 2019-03-31 10:23:13,435 main ERROR No Log4j 2 configuration file found. Using default configuration (logging only errors to the console), or user programmatically provided configurations. Set system property 'log4j2.debug' to show Log4j 2 internal initialization logging. See https://logging.apache.org/log4j/2.x/manual/configuration.html for instructions on how to configure Log4j 2
2019-03-31 10:23:15,380 main ERROR Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")
```
出现错误，显示内存不足
```
$ sudo ./elasticsearch
[sudo] sunnylinux 的密码：
OpenJDK Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
OpenJDK Server VM warning: INFO: os::commit_memory(0x2aa00000, 2080374784, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 2080374784 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /usr/local/elasticsearch-5.6.16/bin/hs_err_pid3717.log
```
由于elasticsearch5.0默认分配jvm空间大小为2g，修改jvm空间分配
https://blog.csdn.net/qq942477618/article/details/53414983
```
# 修改jvm.options
$ pwd
/usr/local/elasticsearch-5.6.16/config

# 修改以下数据
-Xms2g
-Xmx2g

# 修改为512m
-Xms512m
-Xmx512m
```


