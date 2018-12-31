# Linux 系统性能监控
Linux 系统监控一般涉及 进程、CPU、内存、硬盘使用率、硬盘IO、系统负载、网络跟踪 等参数。

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
top     #在运行的进程信息
kill    #杀死进程
nice    #调进程优先级且触发进程
renice  #调现有进程的优先级
```
更多与进程相关的细节 [linux 进程](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%86%85%E6%A0%B8%E3%80%81%E8%BF%9B%E7%A8%8B%E3%80%81%E7%BA%BF%E7%A8%8B.MD#%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86)


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
lscpu
top
htop
vmstat
mpstat
```

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
一般来说， swap 最好不要被使用，尤其 swap 最好不要被使用超过 20% 以上，系统用到swap是因为物理内存不足

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
