# VMware CentOS7虚拟机 网络配置
## 配置方式
### Bridge模式
将虚拟主机直接添加到局域网中，使虚拟机有独立IP，与宿主物理网卡处于平等关系。</br>虚拟机和物理机处于同一个局域网当中。</br>
前提：需要局域网有多余的IP分配给虚拟机。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%201.png)</br>

### NAT模式
VMware根据宿主的真实IP提供Subset IP供虚拟机使用，宿主物理网卡相当于路由器。</br>
原理：</br>
NAT（Network Address Translation，网络地址转换）通过使用少量的公有IP 地址代表较多的私有IP 地址的方式，NAT的出现是为了减缓可用IPv4地址空间的枯竭。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%202.png)

### Host Only 模式
虚拟机不和互联网直接通信，而通过和物理主机通信后，转传给互联网。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%203.png)</br>

## 具体配置
由于不清楚实际使用的无线网络设置状况，所以此处选择NAT的方式比较稳妥。</br>
</br>
默认</br>
VMnet0被设置成为Bridge模式</br>
VMnet1被设置为Host Only模式</br>
VMnet8的默认连接方式为NAT模式</br>
</br>
### 查看
VMware workstation->Edit->Virtual Network Editor
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%204.png)</br>

### 配置过程
设置为NAT模式
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%205.png)</br>
#### 物理计算机配置
打开Win7的“网络和共享中心”，选择“更改适配器设置”，如果VMware workstation安装正确的话，这里应该可以看到VMnet1和VMnet8这两块虚拟网卡。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%206.png)</br>
查看连接有宽带的“本地连接”的属性，顺便在“网络”选项卡下面的“此连接使用下列项目”中将VMware Bridge Protocol勾选上，以便以后选用Bridge模式的时候使用此协议。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%207.png)</br>
因为现时网络用的是无线网络，所以Wireless Network也做和上面一样的处理。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%208.png)</br>
在“共享”选项卡下面“Internet连接共享”中勾选“允许其他网络用户通过此计算机的Internet连接来连接(N)”，并同时在“家庭网络连接(H)”中选择VMnet8。这样设置之后就已经将宿主计算机的网络访问权限与VMnet8共享。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%209.png)</br>




#### VMware上的配置
查看一下VMnet8的网络属性，同样顺便在“网络”选项卡下面将“此连接使用下列项目”中将VMware Bridge Protocol勾选，以便日后Bridge模式使用。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2010.png)</br>
查看TCP/IPv4的属性，可以看到IP地址已经自动变为固定的192.168.137.1，子网掩码已经是255.255.255.0，这些就让其这样设置，暂且不管。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2011.png)</br>
打开VMware workstation->Edit->VMware Network Editor，选择VMnet8，勾选Connect a host virtual network adapter to this network和Use local DHCP service to distribute IP address to VMs。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2012.png)</br>
Subset IP需要与宿主VMnet8的IP在同一地址段，可填写192.168.137.0，Subset Mask应与VMnet8的子网掩码相同，即填写255.255.255.0。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2013.png)</br>

打开DHCP Settings，里面的Start IP address填写192.168.137.1，End IP address填写192.168.137.254，即可在这一IP段内自动分配给虚拟计算机IP地址。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2014.png)</br>
在NAT Settings里面将Getway IP设置为宿主计算机中VMnet8的IP，即192.168.137.1。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2015.png)<br>
#### 虚拟主机上的配置
通过编辑文件修改网络配置</br>
```
# vim /etc/sysconfig/network-scripts/ifcfg-ens33
```
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2016.png)</br>



修改为和以上配置匹配的IP</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20CentOS7%E8%99%9A%E6%8B%9F%E6%9C%BA%20%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE%2017.png)</br>
用/etc/init.d/network restart命令重启网络服务</br>
```
# /etc/init.d/network restart
```
ifconfig 查看虚拟机IP</br>
```
$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.137.100  netmask 255.255.255.0  broadcast 192.168.137.255
        inet6 fe80::fade:7bb1:bd57:33e7  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:1e:07:51  txqueuelen 1000  (Ethernet)
        RX packets 547435  bytes 815635061 (777.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 252465  bytes 17140380 (16.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 19  base 0x2000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 378  bytes 36486 (35.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 378  bytes 36486 (35.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
ping 测试
```
$ ping www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.
64 bytes from www.baidu.com (14.215.177.38): icmp_seq=1 ttl=56 time=151 ms
64 bytes from www.baidu.com (14.215.177.38): icmp_seq=2 ttl=56 time=8.42 ms
64 bytes from www.baidu.com (14.215.177.38): icmp_seq=3 ttl=56 time=6.91 ms
64 bytes from www.baidu.com (14.215.177.38): icmp_seq=4 ttl=56 time=7.48 ms
```
