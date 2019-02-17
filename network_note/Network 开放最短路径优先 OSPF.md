# OSPF 开放最短路径优先
OSPF 是一种路由选择协议，基于狄克斯特拉算法 Dijkstra's Algorithm 创建最短路径树，再得到最佳路径填充路由选择表。</br>
OSPF 没有跳数限制，可创建区域和自主系统，开放标准，适用于多个厂商。</br>
OSPF 区域0必不可少，其他区域都应该连到该区域，将其他区域连接到AS主干区域的路由为“区域边界路由 ABR Area Border Router”，ABR至少有一个接口在区域0内。</br>
[关于 狄克斯特拉算法 Dijkstra's Algorithm 的 python 实现](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E7%AE%97%E6%B3%95%EF%BC%9A%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E7%AE%97%E6%B3%95%E3%80%81%E7%8B%84%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95%20.MD#%E7%8B%84%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95-dijkstras-algorithm)</br>
## OSPF术语
* RID 路由ID ：所有环回接口loopback中选择最大的一个，如果没有设loopback则使用物理ip最大的一个作RID
* 邻居关系：OSPF 只接受和它建立了邻居关系的路由的共享路由，并非所有邻居路由都会建立邻居关系
* 指定路由 DR：广播网络中，所有路由都会将自己的路由信息发送给DR，DR将这些信息整合后共享给所有路由（LSA更新）。优先级高的路由会被选为DR，平级的话选RID大的做DR
* 备用指定路由 BDR：BDR接收路由选择更新但不广播
* OSPF 区域：在同一区域中的所有路由的接口区域ID相同，在同一区域中所有路由的拓扑表相同
* 点到点：物理意义上的两个物理路由接口连起来，逻辑上的两个远程路由接口连起来，点到点不需要DR和BDR
* 点到多点：路由通过单个接口连接多台远程路由
* OSPF 度量值：成本，一般是接口带宽
* OSPF 进程：最小为1，在单台路由上使用OSPF将多个AS连起来的时候会使用多个进程

## OSPF 工作原理
操作步骤：
* 初始化邻居关系
* LSA更新
* 计算最短路径 SPF树 
* OSPF路由将 SPF插入到路由表中

## OSPF 配置
网络拓扑图如下</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/network_ospf.png)</br>
</br>
Router 配置</br>
```
# Lab_A 配置

Lab_A>enable
Lab_A#config t
Lab_A#config terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
Lab_A(config)#int s2/0
Lab_A(config-if)#ip addr
Lab_A(config-if)#ip address 172.16.10.1 255.255.255.0
Lab_A(config-if)#no shut

%LINK-5-CHANGED: Interface Serial2/0, changed state to down
Lab_A(config-if)#router ospf 1
Lab_A(config-router)#network 172.16.10.0 0.0.0.255 area 0

Lab_A(config-router)#int f1/0
Lab_A(config-if)#ip add
Lab_A(config-if)#ip address 172.16.30.1 255.255.255.0
Lab_A(config-if)#no shut

Lab_A(config-router)#int f1/0
Lab_A(config-if)#ip add
Lab_A(config-if)#ip address 172.16.30.1 255.255.255.0
Lab_A(config-if)#no shut

Lab_A(config)#router ospf 1
Lab_A(config-router)#network 172.16.30.0 0.0.0.255 area 0
```
```
# Lab_B 路由配置
Router>enable
Router#config t
Router#config terminal 
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Lab_B
Lab_B(config)#int s2/0 
Lab_B(config-if)#ip addr
Lab_B(config-if)#ip address 172.16.10.2 255.255.255.0
Lab_B(config-if)#no shut

Lab_B(config-if)#
%LINK-5-CHANGED: Interface Serial2/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial2/0, changed state to up

Lab_B(config-if)#router ospf 1
Lab_B(config-router)#network 172.16.10.0 0.0.0.255 area 0 
Lab_B(config-router)#
00:04:03: %OSPF-5-ADJCHG: Process 1, Nbr 172.16.10.1 on Serial2/0 from LOADING to FULL, Loading Done

Lab_B(config-router)#int s3/0
Lab_B(config-if)#ip addr
Lab_B(config-if)#ip address 172.16.20.1 255.255.255.0
Lab_B(config-if)#no shut

%LINK-5-CHANGED: Interface Serial3/0, changed state to down
Lab_B(config-if)#router ospf 1
Lab_B(config-router)#network 172.16.20.0 0.0.0.255 area 0
```
```
# Lab_C 路由配置
Router>enable
Router#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int s2/0
Router(config-if)#ip addr
Router(config-if)#ip address 172.16.20.2 255.255.255.0
Router(config-if)#no shut
%LINK-5-CHANGED: Interface Serial2/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial2/0, changed state to up

Router(config-if)#router ospf 1
Router(config-router)#network 172.16.20.0 0.0.0.255 area 0
Router(config-router)#
01:55:08: %OSPF-5-ADJCHG: Process 1, Nbr 172.16.10.2 on Serial2/0 from LOADING to FULL, Loading Done

Router(config-router)#int f1/0
Router(config-if)#ip addr
Router(config-if)#ip address 172.16.40.1 255.255.255.0
Router(config-if)#no shut

Router(config-if)#
%LINK-5-CHANGED: Interface FastEthernet1/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0, changed state to up

Router(config-if)#router ospf 1
Router(config-router)#network 172.16.40.0 0.0.0.255 area 0
```
主机配置
```
# Laptop5 配置
ip address 172.16.30.2
subnet mask 255.255.255.0
default gateway 172.16.30.1

# Laptop7 配置
ip address 172.16.40.2
subnet mask 255.255.255.0
default gateway 172.16.40.1
```
ping 测试
```
# Laptop5 ping Laptop7

PC>ipconfig

FastEthernet0 Connection:(default port)
Link-local IPv6 Address.........: FE80::2E0:8FFF:FE44:93D
IP Address......................: 172.16.30.2
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 172.16.30.1

PC>ping 172.16.40.2

Pinging 172.16.40.2 with 32 bytes of data:

Reply from 172.16.40.2: bytes=32 time=2ms TTL=125
Reply from 172.16.40.2: bytes=32 time=3ms TTL=125
Reply from 172.16.40.2: bytes=32 time=4ms TTL=125
Reply from 172.16.40.2: bytes=32 time=3ms TTL=125

Ping statistics for 172.16.40.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 2ms, Maximum = 4ms, Average = 3ms
```
## 查看OSPF运行情况
```
# 查邻居
Lab_A>sh ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
172.16.10.2       0   FULL/  -        00:00:37    172.16.10.2     Serial2/0

# 查路由表
Lab_A>sh ip route
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     172.16.0.0/24 is subnetted, 4 subnets
C       172.16.10.0 is directly connected, Serial2/0
O       172.16.20.0 [110/128] via 172.16.10.2, 00:16:00, Serial2/0
C       172.16.30.0 is directly connected, FastEthernet1/0
O       172.16.40.0 [110/129] via 172.16.10.2, 00:13:31, Serial2/0

# 查OSPF信息
Lab_A>sh ip protocols 

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 172.16.10.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    172.16.10.0 0.0.0.255 area 0
    172.16.30.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    172.16.10.1          110      00:23:07
    172.16.10.2          110      00:18:16
    172.16.20.2          110      00:16:47
  Distance: (default is 110)
  
Lab_A>sh ip ospf int f1/0
FastEthernet1/0 is up, line protocol is up
  Internet address is 172.16.30.1/24, Area 0
  Process ID 1, Router ID 172.16.10.1, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 172.16.10.1, Interface address 172.16.30.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:01
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
```
```
Lab_B>sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
172.16.10.1       0   FULL/  -        00:00:37    172.16.10.1     Serial2/0
172.16.20.2       0   FULL/  -        00:00:30    172.16.20.2     Serial3/0

Lab_B>sh ip protocols 

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 172.16.10.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    172.16.10.0 0.0.0.255 area 0
    172.16.20.0 0.0.0.255 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    172.16.10.1          110      00:01:50
    172.16.10.2          110      00:27:00
    172.16.20.2          110      00:25:31
  Distance: (default is 110)

Lab_B>sh ip ospf int s3/0
Serial3/0 is up, line protocol is up
  Internet address is 172.16.20.1/24, Area 0
  Process ID 1, Router ID 172.16.10.2, Network Type POINT-TO-POINT, Cost: 64
  Transmit Delay is 1 sec, State POINT-TO-POINT, Priority 0
  No designated router on this network
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:04
  Index 2/2, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1 , Adjacent neighbor count is 1
    Adjacent with neighbor 172.16.20.2
  Suppress hello for 0 neighbor(s)
```
因为在点到点链路上不会选举DR和BDR，所以状态为 FULL/  -  和 No designated router on this network、No backup designated router on this network
