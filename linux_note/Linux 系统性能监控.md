# Linux 系统性能监控
Linux 系统监控一般涉及 进程、CPU、内存、硬盘使用率、硬盘IO、系统负载、网络跟踪 等参数。
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/2030854783.jpg)
## 系统基本信息
列出系统内核版本、硬件平台、CPU类型
```
$ uname -a
Linux centOSlearning.SharonLi 3.10.0-862.2.3.el7.centos.plus.i686 #1 SMP Wed May 9 18:52:21 UTC 2018 i686 i686 i386 GNU/Linux
```

## 进程
### 进程相关命令
```
ps -l   #列出所有与当前bash相关的进程信息
ps aux  #列出系统中所有的进程信息
ps -lA  #同上
pstree  #进程树
pidof   #找出某个正在执行的进程的pid
top     #在运行的进程信息
kill    #杀死进程
nice    #调进程优先级且触发进程
renice  #调现有进程的优先级
```
更多与进程相关的细节 [linux 进程](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%86%85%E6%A0%B8%E3%80%81%E8%BF%9B%E7%A8%8B%E3%80%81%E7%BA%BF%E7%A8%8B.MD#%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86)
```
$ pidof python3
4226
```

### 查和进程相关的文件的命令
```
lsof #查看进程打开的文件
lsof -i : 80  #找对应端口被哪个进程占用
lsof  /tmp/1.txt #找对应文件被哪个进程占用
lsof -p pid  #找pid对应进程所打开的文件
```
它常用于以列表形式显示所有打开的文件和进程，包括磁盘文件，网络套接字，管道，设备和进程</br>
情形：无法挂载磁盘和显示正在使用或者打开某个文件的错误时，查看谁正在使用</br>
```
$ lsof -p 3639
COMMAND  PID       USER   FD   TYPE DEVICE  SIZE/OFF     NODE NAME
python3 3639 sunnylinux  cwd    DIR  253,2      4096       67 /home/sunnylinux
python3 3639 sunnylinux  rtd    DIR  253,0       247       64 /
python3 3639 sunnylinux  txt    REG  253,0   9674412  2572636 /usr/local/python3/bin/python3.7
python3 3639 sunnylinux  mem    REG  253,0    136312 17225343 /usr/lib/libtinfo.so.5.9
（省略很多输出）

```


## CPU
CPU相关的命令
```
lscpu    #输出CPU信息
top      #动态输出各种和CPU相关的信息
htop     #高级的交互式实时linux进程监控工具，和top相似，但更友好, 还支持鼠标
vmstat   #检测系统资源变化，系统繁忙的时候用
mpstat   #监控到cpu的一些统计信息，在多核cpu的系统里不但能够查看所有cpu的平均状况信息，而且能够查看特定的cpu的信息
```
### lscpu
```
$ lscpu
Architecture:          i686
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
座：                 1
厂商 ID：           AuthenticAMD
CPU 系列：          16
型号：              6
型号名称：        AMD Turion(tm) II Dual-Core Mobile M520
步进：              2
CPU MHz：             2294.279
BogoMIPS：            4588.55
L1d 缓存：          64K
L1i 缓存：          64K
L2 缓存：           512K
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm 3dnowext 3dnow constant_tsc art tsc_reliable nonstop_tsc pni cx16 x2apic popcnt hypervisor lahf_lm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw retpoline_amd ibp_disable vmmcall

```
### 查进程和CPU top
[top的使用](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%86%85%E6%A0%B8%E3%80%81%E8%BF%9B%E7%A8%8B%E3%80%81%E7%BA%BF%E7%A8%8B.MD#%E5%8A%A8%E6%80%81%E6%9F%A5%E8%AF%A2%E8%BF%9B%E7%A8%8B%E7%8A%B6%E6%80%81)</br>
```
top - 09:57:39 up 15 min,  1 user,  load average: 0.04, 0.14, 0.25
Tasks: 159 total,   2 running, 157 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.7 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1027752 total,   336128 free,   342700 used,   348924 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.   509800 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 2111 sunnyli+  20   0   10260   1900   1340 R  0.7  0.2   0:00.27 top
  385 root      20   0       0      0      0 S  0.3  0.0   0:00.55 xfsaild/dm-0
  846 root      20   0   31224   4264   3516 S  0.3  0.4   0:03.20 vmtoolsd
 1671 mysql     20   0  589928  72624   6032 S  0.3  7.1   0:02.18 mysqld
 2054 root      20   0       0      0      0 S  0.3  0.0   0:00.07 kworker/0:0
    1 root      20   0   27052   5748   3956 S  0.0  0.6   0:05.20 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.19 ksoftirqd/0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
    9 root      20   0       0      0      0 R  0.0  0.0   0:01.97 rcu_sched
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain
   11 root      rt   0       0      0      0 S  0.0  0.0   0:00.72 watchdog/0
   13 root      20   0       0      0      0 S  0.0  0.0   0:00.01 kdevtmpfs
   14 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 netns
   15 root      20   0       0      0      0 S  0.0  0.0   0:00.00 khungtaskd
   16 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 writeback
   17 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kintegrityd
   18 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset
   19 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kblockd
```
###　vmstat
vmstat 可动态地检测系统资源变化，包括CPU/内存/磁盘输入输出状态
```
vmstat 参数
-a：显示活跃和非活跃内存
-m：显示slabinfo
-n：只在开始时显示一次各字段名称。
-s：显示内存相关统计信息及多种系统活动数量。
delay：刷新时间间隔。如果不指定，只显示一条结果。
count：刷新次数。如果不指定刷新次数，但指定了刷新时间间隔，这时刷新次数为无穷。
-d：显示各个磁盘相关统计信息。
-S：使用指定单位显示。参数有 k 、K 、m 、M ，分别代表1000、1024、1000000、1048576字节（byte）。默认单位为K（1024 bytes）
-V：显示vmstat版本信息。
-p：显示指定磁盘分区统计信息
-D：显示磁盘总体信息

#查看CPU内存等信息，可设延迟时间以及检测次数
$ vmstat -a
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 3  0  44992  75376 447736 402724    1    5   327    10  131  133  2  2 94  1  0
#延迟1秒查3次
$ vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 4  0  45248  84040      0 314508    0    5   320    10  131  134  2  3 94  1  0
 0  0  45248  84780      0 314508    0    0     0     0  577  280 38 18 44  0  0
 0  0  45248  84740      0 314508    0    0     0     0  608  342 32 15 54  0  0
```
若si/so很大则表示磁盘和内存进程交互，系统性能差</br>
磁盘读写bi和bo，数值很高则磁盘读写繁忙</br>
system in 每秒被中断的进程次数，cs 每秒进行事件切换的次数，数值大代表系统和接口通信频繁</br>
```
#系统从启动初期开始到现在的forks数量
$ vmstat -fs
         4043 forks

#系统上所有磁盘的读写状态
$ vmstat -d
disk- ------------reads------------ ------------writes----------- -----IO------
       total merged sectors      ms  total merged sectors      ms    cur    sec
fd0        0      0       0       0      0      0       0       0      0      0
sda    47768   1054 5704387 1450438   4758  11644  173909   40541      0    373
sr0        0      0       0       0      0      0       0       0      0      0
dm-0   45681      0 5508057 1378010   4577      0   69385   22299      0    356
dm-1    1198      0   13288   35328  11332      0   90656  682753      0     19
dm-2    1523      0  160816   72081    490      0    9772    3045      0     29
```
mpstat 可用于判断io瓶颈
```
$ mpstat
Linux 3.10.0-862.2.3.el7.centos.plus.i686 (centOSlearning.SharonLi) 	2018年12月31日 	_i686_	(1 CPU)

17时58分50秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
17时58分50秒  all    1.64    0.19    2.19    1.04    0.00    0.04    0.00    0.00    0.00   94.91

```
%iowait列，CPU等待I/O操作所花费的时间，这个值持续很高通常可能是I/O瓶颈所导致的，通过这个参数可以比较直观的看出当前的I/O操作是否存在瓶颈</br>
%idle 的值高但系统响应慢时，有可能是 CPU 等待分配内存，此时应加大内存容量</br>
%idle 的值持续低于1，则系统的 CPU 处理能力相对较低，表明系统中最需要解决的资源是 CPU</br>
## 内存
free 查看系统内存状况 <br>
-b 选项 以字节为单位 显示 内存总和; -k 选项 (缺省的) 以 KB 为单位 显示;<br>
-m 选项 以 MB 为单位.<br>
-t 选项 显示 一个 总计行.<br>
-o  选项  禁止  "buffer  adjusted" 行的显示. 除非 指定 free 从 (相应的)已用/未用的 内存 减去/加上 缓冲区内存.<br>
-s 使 free 以 delay  秒为间隔,  连续抽样显示.  delay  可以设置成浮点数,它用 usleep(3) 做 微秒级 延迟.<br>
-V 显示版本信息.<br>

```
$ free
              total        used        free      shared  buff/cache   available
Mem:        1027676      698396       67884        4768      261396      125896
Swap:       1048572       35840     1012732

```
一般来说， swap 最好不要被使用，尤其 swap 最好不要被使用超过 20% 以上，系统用到swap是因为物理内存不足</br>
</br>
Linux中的交换分区的大小分配推荐法则</br>
曾经，有人推荐使用物理内存1/2、1倍、2倍的容量作为SWAP分区的大小。随着计算机内容容量的增大，如果有16G内存的机器，分个16G的Swap空间，似乎感觉有点浪费磁盘空间，而且多数情况下用处也并不大。在Linux系统，我们可以参照Redhat公司为RHEL5、RHEL6推荐的SWAP空间的大小划分原则，在你没有其他特别需求时，可以作为很好的参考依据。</br>
```
内存小于4GB时，推荐不少于2GB的swap空间；
内存4GB~16GB，推荐不少于4GB的swap空间；
内存16GB~64GB，推荐不少于8GB的swap空间；
内存64GB~256GB，推荐不少于16GB的swap空间。
```
## 硬盘使用率和硬盘IO
### 硬盘使用
```
df  #查看硬盘使用空间
dd
```
关于[df 使用](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%88%A0%E9%99%A4%E5%A4%9A%E4%BD%99%E6%97%A0%E7%94%A8%E5%88%86%E5%8C%BA.MD#df-%E6%8A%A5%E5%91%8A%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%A3%81%E7%9B%98%E7%A9%BA%E9%97%B4%E7%9A%84%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5)

### 硬盘IO
以下两命令需要安装
```
iotop  #监控linux磁盘I/O, 用于查找大量使用磁盘读写进程的时候
iostat #查看存储设备输入和输出状态统计的工具，用来追踪存储设备的性能问题；包括设备，磁盘，NFS远程磁盘
```

## 系统负载
```
uptime  #显示目前系统开机多久和系统负载（和top相似）
```
```
$ uptime
18:46:27 up  1:47,  2 users,  load average: 0.08, 0.05, 0.08
```

## 网络跟踪
和网络相关的命令
```
netstat   #用于监控进出网络的包和网络接口统计的命令行工具
tcpdump   #用于捕捉或过滤网络上指定接口上接收或者传输的TCP/IP包，作用和 wireshark 相似，需要额外安装

```

### netstat
```
netstat [选项]
-a (all)：显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED），也包括监听连接请求（LISTENING）的那些连接，断开连接（CLOSE_WAIT）或者处于联机等待状态的（TIME_WAIT）等
-t (tcp)：显示tcp相关选项
-u (udp)：仅显示udp相关选项
-n ：拒绝显示别名，能显示数字的全部转化成数字。
-l ：仅列出有在 Listen (监听) 的服务状态
-p ：显示建立相关链接的程序名
-r ：显示路由信息，路由表，除了显示有效路由外，还显示当前有效的连接
-e ：显示扩展信息，例如uid等
-s ：按各个协议进行统计
-c ：每隔一个固定时间，执行该netstat命令。
-v ：显示当前的有效连接，与-n选项类似
-I ：显示自动匹配接口的信息
-e ：显示关于以太网的统计数据。它列出的项目包括传送的数据报的总字节数、错误数、删除数、数据报的数量和广播的数量。这些统计数据既有发送的数据报数量，有接收的数据报数量。这个选项可以用来统计一些基本的网络流量。
提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到
```
```
$ netstat

Active Internet connections (w/o servers)  #网络相关
# Proto 协议(tcp/udp)、Recv-Q 非用户进程连接到此socket的字节数、Send-Q 非远程主机发送过来的aknowledged字节数、State 连接状态（established/listen）
Proto Recv-Q Send-Q Local Address           Foreign Address         State    

Active UNIX domain sockets (w/o servers)  #与本机进程相关
# RefCnt 连接到此socket的进程数、Flags 连接标式、 Type 访问类型（确认STEAM/不需确认DGRAM)、State connected则为多个进程已连接
Proto RefCnt Flags       Type       State         I-Node   Path
unix  5      [ ]         DGRAM                    7432     /run/systemd/journal/socket
unix  22     [ ]         DGRAM                    7434     /dev/log
unix  2      [ ]         DGRAM                    12127    /run/systemd/shutdownd
unix  3      [ ]         DGRAM                    7409     /run/systemd/notify
unix  2      [ ]         DGRAM                    7411     /run/systemd/cgroups-agent
unix  3      [ ]         STREAM     CONNECTED     16597    
unix  3      [ ]         STREAM     CONNECTED     37993    @/tmp/.X11-unix/X0

```
查看目前主机所提供的服务和对应的pid（开启的接口service ports）</br>
查到pid后可用kill关闭该网络服务</br>
```
端口号
80: WWW
22: ssh
21: ftp
25: mail
111: RPC(远端程序呼叫)
631: CUPS(打印服务功能)

$ netstat -tlnp  
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 ::1:631                 :::*                    LISTEN      -                   
tcp6       0      0 ::1:25                  :::*                    LISTEN      -                   

$ sudo netstat -tlnp
[sudo] sunnylinux 的密码：
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1672/mysqld         
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1183/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1178/cupsd          
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1676/master         
tcp6       0      0 :::22                   :::*                    LISTEN      1183/sshd           
tcp6       0      0 ::1:631                 :::*                    LISTEN      1178/cupsd          
tcp6       0      0 ::1:25                  :::*                    LISTEN      1676/master         
#进程后面有个d说明这个进程是启动对应daemon的进程
```
查看路由表
```
$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         Tommy-PC.mshome 0.0.0.0         UG        0 0          0 ens33
192.168.137.0   0.0.0.0         255.255.255.0   U         0 0          0 ens33

```
查看网络接口
```
$ netstat -i
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
ens33     1500     2034      0      0 0           728      0      0      0 BMRU
lo       65536       94      0      0 0            94      0      0      0 LRU
```
