# OSPF 开放最短路径优先
OSPF 是一种路由选择协议，基于狄克斯特拉算法 Dijkstra's Algorithm 算法创建最短路径树，再得到最佳路径填充路由选择表。</br>
OSPF 没有跳数限制，可创建区域和自主系统，开放标准，适用于多个厂商。</br>
OSPF 区域0必不可少，其他区域都应该连到该区域，将其他区域连接到AS主干区域的路由为“区域边界路由 ABR Area Border Router”，ABR至少有一个接口在区域0内。</br>
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
Router 配置
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
Lab_A(config-if)#io addr
Lab_A(config-if)#io address
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
