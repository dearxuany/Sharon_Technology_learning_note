# 端口 port
## 定义
从OSI七层的划分来看，网络层上每部主机都会分配有一个IP，这个网络上的其他主机会通过这个IP定位到这部主机，然后给这部主机发送信息。</br>
可是对于主机本身来说，主机上多个程序都想通过网络接受或发送信息，但网络分配给主机的IP只有一个，那主机如何分辨接收到的信息是应该发送给主机上的哪个程序？又如何分辨主机上的程序有序地通过网络发送信息呢？</br>
由此，出现了端口port这个概念，端口的设计让计算机可以并发运行多个网络程序且保证多个程序之间不相互干扰。在进行网络通讯交换时，网络上的主机可通过IP地址查找到其他主机，然后通过端口标识接收方主机上的一个唯一的程序。</br>
注意：此处谈论的是软件层面的端口，要和物理端口作区分</br>

## 规范
在硬件上规定，端口的号码必须位于0-65535之间，每个端口唯一的对应一个网络程序，一个网络程序可以使用多个端口；</br>
通信双方的端口并不一定是对应的，如服务端用80，而客户端是可以用3457的；</br>
根据提供服务类型的不同，端口分为两种，一种是TCP端口，一种是UDP端口，但是tcp和udp使用的端口都不是固定的，是可以被选择的，同时同一端口号可以被UDP和TCP同时使用而不冲突。
有些端口根据业界规范被固定做特殊的用途，这些端口号不能被随便使用，协议里面低于1024的端口都有确切的定义，它们对应着因特网上常见的一些服务。</br>
### 周知端口 Well Known Ports
周知端口是众所周知的端口号，范围从0到1023，其中80端口分配给WWW服务，加密的https协议所使用的端口为443，21端口分配给FTP服务等；</br>
从0到1023，它们紧密绑定（binding）于一些服务。通常这些端口的通讯明确表明了某种服务的协议；</br>
每一个标准服务器都拥有一个全局公认的端口，即是所有提供同种网络服务的服务器都会使用同一个端口号，以便减少沟通成本。</br>
### 注册端口 Registered Ports
端口1024到49151，分配给用户进程或应用程序；</br>
这些进程主要是用户选择安装的一些应用程序，而不是已经分配好了公认端口的常用程序；</br>
这些端口在没有被服务器资源占用的时候，可以被用户端动态选用为源端口。</br>
### 动态端口 Dynamic Ports
动态端口的范围是从49152到65535一般不固定分配某种服务，而是动态分配。</br>
程需要访问传输层服务时，向本地操作系统提出申请，操作系统返回本地唯一的端口号，进程再通过合适的系统调用，将自己和该端口连接起来（binding，绑定）。</br>

## 常用端口号
```
http    80或8080
https   443
ftp     21
telnet  23
smpt    25
ssh     22
Mysql   3306 
mongodb 2719
```
## 端口重定向
实现重定向是为了隐藏公认的默认端口，降低受破坏率，提高安全性。

## Linux 端口查询
查询服务对应端口
```
sunnylinux@centOSlearning system]$ cat /etc/services|grep https
https           443/tcp                         # http protocol over TLS/SSL
https           443/udp                         # http protocol over TLS/SSL
https           443/sctp                        # http protocol over TLS/SSL
oob-ws-https    664/tcp                 # DMTF out-of-band secure web services management protocol
llsurfup-https  1184/tcp                # LL Surfup HTTPS
llsurfup-https  1184/udp                # LL Surfup HTTPS
compaq-https    2381/tcp                # Compaq HTTPS
compaq-https    2381/udp                # Compaq HTTPS
wap-push-https  4036/tcp                # WAP Push OTA-HTTP secure
wap-push-https  4036/udp                # WAP Push OTA-HTTP secure
appserv-https   4849/tcp                # App Server - Admin HTTPS
appserv-https   4849/udp                # App Server - Admin HTTPS
wbem-https      5989/tcp                # WBEM CIM-XML (HTTPS)
wbem-https      5989/udp                # WBEM CIM-XML (HTTPS)
wbem-exp-https  5990/tcp                # WBEM Export HTTPS
wbem-exp-https  5990/udp                # WBEM Export HTTPS
sun-sr-https    6443/tcp                # Service Registry Default HTTPS Domain
sun-sr-https    6443/udp                # Service Registry Default HTTPS Domain
plysrv-https    6771/tcp                # PolyServe https
plysrv-https    6771/udp                # PolyServe https
smc-https       6789/tcp                # SMC-HTTPS
smc-https       6789/udp                # SMC-HTTPS
oracleas-https  7443/tcp                # Oracle Application Server HTTPS
oracleas-https  7443/udp                # Oracle Application Server HTTPS
sun-user-https  7677/tcp                # Sun App Server - HTTPS
sun-user-https  7677/udp                # Sun App Server - HTTPS
synapse-nhttps  8243/tcp                # Synapse Non Blocking HTTPS
synapse-nhttps  8243/udp                # Synapse Non Blocking HTTPS
pcsync-https    8443/tcp                # PCsync HTTPS
pcsync-https    8443/udp                # PCsync HTTPS
https-wmap      8991/tcp                # webmail HTTPS service
https-wmap      8991/udp                # webmail HTTPS service
armcenterhttps  9295/tcp                # ARMCenter https Service
armcenterhttps  9295/udp                # ARMCenter https Service
tungsten-https  9443/tcp                # WSO2 Tungsten HTTPS
tungsten-https  9443/udp                # WSO2 Tungsten HTTPS
amt-soap-https  16993/tcp               # Intel(R) AMT SOAP/HTTPS
amt-soap-https  16993/udp               # Intel(R) AMT SOAP/HTTPS
commtact-https  20003/tcp               # Commtact HTTPS
commtact-https  20003/udp               # Commtact HTTPS

[sunnylinux@centOSlearning system]$ cat /etc/services|grep http
#       http://www.iana.org/assignments/port-numbers
http            80/tcp          www www-http    # WorldWideWeb HTTP
http            80/udp          www www-http    # HyperText Transfer Protocol
http            80/sctp                         # HyperText Transfer Protocol
https           443/tcp                         # http protocol over TLS/SSL
https           443/udp                         # http protocol over TLS/SSL
https           443/sctp                        # http protocol over TLS/SSL

[sunnylinux@centOSlearning system]$ cat /etc/services|grep ssh
ssh             22/tcp                          # The Secure Shell (SSH) Protocol
ssh             22/udp                          # The Secure Shell (SSH) Protocol
x11-ssh-offset  6010/tcp                        # SSH X11 forwarding offset
ssh             22/sctp                 # SSH
sshell          614/tcp                 # SSLshell
sshell          614/udp                 #       SSLshell
netconf-ssh     830/tcp                 # NETCONF over SSH
netconf-ssh     830/udp                 # NETCONF over SSH
sdo-ssh         3897/tcp                # Simple Distributed Objects over SSH
sdo-ssh         3897/udp                # Simple Distributed Objects over SSH
snmpssh         5161/tcp                # SNMP over SSH Transport Model
snmpssh-trap    5162/tcp                # SNMP Notification over SSH Transport Model
tl1-ssh         6252/tcp                # TL1 over SSH
tl1-ssh         6252/udp                # TL1 over SSH
ssh-mgmt        17235/tcp               # SSH Tectia Manager
ssh-mgmt        17235/udp               # SSH Tectia Manager

[sunnylinux@centOSlearning system]$ cat /etc/services|grep mysql
mysql           3306/tcp                        # MySQL
mysql           3306/udp                        # MySQL
mysql-cluster   1186/tcp                # MySQL Cluster Manager
mysql-cluster   1186/udp                # MySQL Cluster Manager
mysql-cm-agent  1862/tcp                # MySQL Cluster Manager Agent
mysql-cm-agent  1862/udp                # MySQL Cluster Manager Agent
mysql-im        2273/tcp                # MySQL Instance Manager
mysql-im        2273/udp                # MySQL Instance Manager
mysql-proxy     6446/tcp                # MySQL Proxy
mysql-proxy     6446/udp                # MySQL Proxy
```

