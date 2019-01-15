# Python 通过 psutil 对 linux 进行监控
## 系统性能信息模块 psutil
psutil是一个跨平台库，能够轻松实现获取系统运行的进程和系统利用率（包括CPU、内存、磁盘、网络等）信息。它主要应用于系统监控，分析和限制系统资源及进程的管理。它实现了同等命令行工具提供的功能，如ps、top、lsof、netstat、ifconfig、who、df、kill、free、nice、ionice、iostat、iotop、uptime、pidof、tty、taskset、pmap等。

## CPU
```
>>> import psutil
# 逻辑cpu数量
>>> psutil.cpu_count()
1
# 物理cpu数量
>>> psutil.cpu_count(logical=False)
1

# cpu整体使用率（从上次调用percent开始计算）
>>> psutil.cpu_percent()
21.8
# cpu使用率，指定时间间隔，输出为每个cpu的使用率
>>> psutil.cpu_percent(interval=3,percpu=True)
[21.1]
>>> psutil.cpu_percent(interval=2,percpu=True)
[36.9]

# cpu time: CPU时间反映CPU全速工作时完成该进程所花费的时间，也是内核时间（kernel time)
>>> psutil.cpu_times()
scputimes(user=370.63, nice=1.17, system=329.08, idle=4795.49, iowait=54.96, irq=0.0, softirq=4.96, steal=0.0, guest=0.0, guest_nice=0.0)
# 间隔为1，刷新3次，输出每个cpu的cpu time
>>> for x in range(3):
...     psutil.cpu_percent(interval=1, percpu=True)
... 
[29.0]
[46.5]
[32.7]
>>> psutil.cpu_times_percent()
scputimes(user=4.7, nice=0.0, system=3.8, idle=91.2, iowait=0.2, irq=0.0, softirq=0.1, steal=0.0, guest=0.0, guest_nice=0.0)

# cpu的统计信息，包括上下文切换、中断、软中断和系统调用的次数
>>> psutil.cpu_stats()
scpustats(ctx_switches=1253621, interrupts=1367234, soft_interrupts=1056908, syscalls=0)
```
## 内存
virtual_memory 以命名元组的形式返回内存使用情况，包括总内存、可用内存、内存利用率、 buffer和cached等。</br>
除了内存利用 ，其他字段都以字节为单位返回，所以为了可读性可能需要写个函数来改一下输出的单位，改成常见的 G/K/M 等等。</br>
注意：total（内存总数）、used（已使用的内存数）、free（空闲内存数）、buffers（缓冲使用数）、cache（缓存使用数）、swap（交换分区使用数）
```
>>> psutil.virtual_memory()
svmem(total=1052327936, available=95428608, percent=90.9, used=785432576, free=86507520, active=359854080, inactive=368095232, buffers=0, cached=180387840, shared=5054464, slab=96137216)
```
swap分区
```
>>> psutil.swap_memory()
sswap(total=1073737728, used=15474688, free=1058263040, percent=1.4, sin=319488, sout=15335424)
```
## 硬盘
disk_partitions 返回所有已 经挂载的磁盘，以命名元组的形式返回。命名元组包含磁盘名称、挂载点、文件系统类型等信息。
```
>>> psutil.disk_partitions()
[sdiskpart(device='/dev/mapper/centos-root', mountpoint='/', fstype='xfs', opts='rw,seclabel,relatime,attr2,inode64,noquota'), sdiskpart(device='/dev/sda2', mountpoint='/boot', fstype='xfs', opts='rw,seclabel,relatime,attr2,inode64,noquota'), sdiskpart(device='/dev/mapper/centos-home', mountpoint='/home', fstype='xfs', opts='rw,seclabel,relatime,attr2,inode64,noquota')]
```
disk_ usage 获取磁盘的使用情况，包括磁盘的容量 已经使用的磁盘容量、磁盘的空间利用率等。
```
>>> psutil.disk_usage('/')
sdiskusage(total=25759318016, used=11995742208, free=13763575808, percent=46.6)
>>> psutil.disk_usage('/').percent
46.6
```
disk io counters 以命名元组的形式返回磁盘 io 统计信息，包括读的次数、写的次数，读字节数、写字节数等，省去了解析 /proc/diskstats 文件。
```
>>> psutil.disk_io_counters()
sdiskio(read_count=48431, write_count=16623, read_bytes=1842754048, write_bytes=199751680, read_time=1977299, write_time=900662, read_merged_count=231, write_merged_count=4561, busy_time=619394)
# 每个磁盘，返回一个字典
>>> psutil.disk_io_counters(perdisk=True)
```
## 网络
net_io_counter 函数以命名元组的形式返回了每块网卡的网络 io 统计信息，包括收发字节数、 收发包的数量、出错情况与删包情况，与查/proc/net/dev有相同效果。
```
>>> psutil.net_io_counters()
snetio(bytes_sent=248739, bytes_recv=3720645, packets_sent=2998, packets_recv=3729, errin=1, errout=0, dropin=0, dropout=0)
>>> psutil.net_io_counters(pernic=True)
{'lo': snetio(bytes_sent=10128, bytes_recv=10128, packets_sent=96, packets_recv=96, errin=0, errout=0, dropin=0, dropout=0), 
'ens33': snetio(bytes_sent=238611, bytes_recv=3710793, packets_sent=2902, packets_recv=3636, errin=1, errout=0, dropin=0, dropout=0)}

```
net_connections 以列形式返回每个网络连接的详细信息，可以使用该函数查看网络连接状态，统计连接个数及处于特定状态的网络连接个数。
```
>>> psutil.net_connections()
```
net_if_addrs 以字典的形式返回网卡的配置信息，包括 ip 地址或 ma 地址、子网掩码、广播地址
```
>>> psutil.net_if_addrs()
```
net_if_stats 返回网卡的详细信息，包括是否启动、通信类型、传输速度与mtu
```
>>> psutil.net_if_stats()
{'lo': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_UNKNOWN: 0>, speed=0, mtu=65536), 
'ens33': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_UNKNOWN: 0>, speed=0, mtu=1500)}

```
## 用户
返回当前登录用户的信息，包括用户名、登录时间、终端与主机信息
```
# 最后一条是ssh远程登录
>>> psutil.users()
[suser(name='sunnylinux', terminal=':0', host='localhost', started=1547456910.0, pid=2186), 
suser(name='sunnylinux', terminal='pts/0', host='localhost', started=1547457023.0, pid=3368), 
suser(name='sunnylinux', terminal='pts/1', host='tommy-pc.mshome.net', started=1547471118.0, pid=4372)]
```
## 系统启动时间
返回格式是时间戳，需要用 datatime 进行格式转换
```
>>> psutil.boot_time()
1547459601.0
```

