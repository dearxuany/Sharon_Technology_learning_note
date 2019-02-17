# 动态主机配置协议 DHCP 原理及配置
动态主机配置协议 DHCP 用于给网络环境中的主机动态的获得IP地址、Gateway地址、DNS服务器地址等信息。</br>
DHCP服务器需要配置的信息：
* 地址池名称
* 子网网络IP（地址池）和掩码
* 默认网关
* DNS的IP
* 租期
* 保留IP
* DHCP中继让DHCP服务器给其他子网分配ip
## DHCP 配置
承接 [路由选择：开放最短路径优先 OSPF 原理与配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/network_note/Network%20%E5%BC%80%E6%94%BE%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84%E4%BC%98%E5%85%88%20OSPF.md)中的网络配置，并在此基础上添加子网和路由选择策略</br>
网络拓扑更新为下图所示：</br>
</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/network_dhcp.jpg)</br>
```
Lab_B>enable
Lab_B#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Lab_B(config)#int f1/0
Lab_B(config-if)#ip addr
Lab_B(config-if)#ip address 172.16.50.1 255.255.255.0
Lab_B(config-if)#no shut

Lab_B(config-if)#router ospf 1
Lab_B(config-router)#network 172.16.50.0 0.0.0.255 area 0 
```
DNS 服务器配置
```
ip address 172.16.50.2
subnet mask 255.255.255.0
default gateway 172.16.50.1
```
DHCP 服务器配置
```
ip address 172.16.50.3
subnet mask 255.255.255.0
default gateway 172.16.50.1
dns server 172.16.50.2

# 默认地址池
poolname serverpool
default gateway 172.16.50.1
dns server 172.16.50.2
start ip 172.16.50.4
subnet 255.255.255.0

```
将pc设置为动态获取ip地址，可以看到和DHCP服务器同一子网的主机已经成功获得ip
```
PC>ipconfig

FastEthernet0 Connection:(default port)
Link-local IPv6 Address.........: FE80::207:ECFF:FE85:36D0
IP Address......................: 172.16.50.4
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 172.16.50.1

PC>ping 172.16.50.3

Pinging 172.16.50.3 with 32 bytes of data:

Reply from 172.16.50.3: bytes=32 time=1ms TTL=128
Reply from 172.16.50.3: bytes=32 time=0ms TTL=128
Reply from 172.16.50.3: bytes=32 time=1ms TTL=128
Reply from 172.16.50.3: bytes=32 time=0ms TTL=128

Ping statistics for 172.16.50.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```
配置DHCP中继，让其他子网也可使用DHCP
```
# Lab_C 配置
Router>enable
Router#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Lab_C
Lab_C(config)#int f1/0
Lab_C(config-if)#ip helper-a
Lab_C(config-if)#ip helper-address 172.16.50.3
```
DHCP 添加地址池
```
poolname 172.16.40.0
default gateway 172.16.40.1
dns server 172.16.50.2
start ip 172.16.40.2
subnet 255.255.255.0
```
```
PC>ipconfig

FastEthernet0 Connection:(default port)
Link-local IPv6 Address.........: FE80::201:C7FF:FE5C:3966
IP Address......................: 172.16.40.3
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 172.16.40.1

PC>ping 172.16.50.3

Pinging 172.16.50.3 with 32 bytes of data:

Reply from 172.16.50.3: bytes=32 time=2ms TTL=126
Reply from 172.16.50.3: bytes=32 time=10ms TTL=126
Reply from 172.16.50.3: bytes=32 time=1ms TTL=126
Reply from 172.16.50.3: bytes=32 time=4ms TTL=126

Ping statistics for 172.16.50.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 10ms, Average = 4ms
```
