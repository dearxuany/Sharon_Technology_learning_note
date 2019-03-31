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
重新启动再次报错
```
[2019-03-31T11:41:56,943][INFO ][o.e.t.TransportService   ] [mojEo8w] publish_address {192.168.137.101:9300}, bound_addresses {192.168.137.101:9300}
[2019-03-31T11:41:57,019][INFO ][o.e.b.BootstrapChecks    ] [mojEo8w] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
[2019-03-31T11:41:57,184][INFO ][o.e.n.Node               ] [mojEo8w] stopping ...
[2019-03-31T11:41:57,314][INFO ][o.e.n.Node               ] [mojEo8w] stopped
[2019-03-31T11:41:57,315][INFO ][o.e.n.Node               ] [mojEo8w] closing ...
[2019-03-31T11:41:57,566][INFO ][o.e.n.Node               ] [mojEo8w] closed
```
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
sudo vim /etc/security/limits.conf 
#添加以下内容，***是启动elk的用户
*** soft nofile 65536 
*** hard nofile 65536 
#用户退出重新登录自动生效
```
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
sudo vim /etc/sysctl.conf
# 添加
vm.max_map_count=262144

# 执行sysctl -p使配置生效
$ sysctl -p
sysctl: permission denied on key 'vm.max_map_count'
$ sudo sysctl -p
vm.max_map_count = 262144
```
[3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
问题原因：因为Centos6不支持SecComp，而ES默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。详见 ：https://github.com/elastic/elasticsearch/issues/22899
```
# 在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```
重新启动 es
```
[2019-03-31T12:26:01,384][INFO ][o.e.n.Node               ] [mojEo8w] starting ...
[2019-03-31T12:26:02,129][INFO ][o.e.t.TransportService   ] [mojEo8w] publish_address {192.168.137.101:9300}, bound_addresses {192.168.137.101:9300}
[2019-03-31T12:26:02,192][INFO ][o.e.b.BootstrapChecks    ] [mojEo8w] bound or publishing to a non-loopback address, enforcing bootstrap checks
[2019-03-31T12:26:05,561][INFO ][o.e.c.s.ClusterService   ] [mojEo8w] new_master {mojEo8w}{mojEo8wYQLCs3HJyNkSvwQ}{XjZ0LYRvRo6W1poiGFUxag}{192.168.137.101}{192.168.137.101:9300}, reason: zen-disco-elected-as-master ([0] nodes joined)
[2019-03-31T12:26:05,742][INFO ][o.e.g.GatewayService     ] [mojEo8w] recovered [0] indices into cluster_state
[2019-03-31T12:26:05,775][INFO ][o.e.h.n.Netty4HttpServerTransport] [mojEo8w] publish_address {192.168.137.101:9200}, bound_addresses {192.168.137.101:9200}
[2019-03-31T12:26:05,775][INFO ][o.e.n.Node               ] [mojEo8w] started

```
