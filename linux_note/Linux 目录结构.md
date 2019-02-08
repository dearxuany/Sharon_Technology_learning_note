# Linux 目录结构
## FHS 标准
[FHS_2.3 标准文档](http://refspecs.linuxfoundation.org/FHS_2.3/fhs-2.3.pdf)</br>
FHS（英文：Filesystem Hierarchy Standard 中文：文件系统层次结构标准），多数 Linux 版本采用这种文件组织形式。
FHS 定义了系统中每个区域的用途、所需要的最小构成的文件和目录同时还给出了例外处理与矛盾处理。</br>
FHS 定义了两层规范:</br>
* 第一层是 / 下面的各个目录应该要放什么文件数据，例如 /etc 应该放置设置文件，/bin 与 /sbin 则应该放置可执行文件等等。
* 第二层则是针对 /usr 及 /var 这两个目录的子目录来定义，例如 /var/log 放置系统日志文件，/usr/share 放置共享数据等等。
## 目录结构
/bin、/etc、/lib、/dev、/sbin 必须放置于根目录下同一分区，/var 建议另开一分区，/usr 没有硬性要求。
### 根目录
```
# 根目录的子目录
[sunnylinux@centOSlearning /]$ tree -L 1
.
├── bin -> usr/bin   # 放置可执行文件的目录，一般用户可用，即是放系统内置日常常用命令的地方，挂载前即可用
├── boot   # 放置开机会使用的文件
├── data
├── db
├── dev   # 设备文件，访问该文件等于访问该设备，常用设备挂载点
├── etc   # 配置文件，一般用户可查阅，root和sudo用户可修改
├── home  # 一般用户的/home
├── lib -> usr/lib   # /bin 执行文件和内核所需要的模块库
├── media  # 可删除设备挂载于此，如软盘、光盘、DVD等
├── mnt    # 暂时设备挂载点
├── opt    # 第三方软件放置目录，和/usr/local相似
├── proc   # 虚拟文件系统，记录系统的内核、进程、外部设备、网络状态，置于内存不占硬盘位置
├── root   # root 的/home
├── run
├── sbin -> usr/sbin  # root 和 sudo用户 才能执行的命令，如还原、开机、关机等
├── srv   # service所写，放置系统服务启动后的数据
├── sys   # 和/proc相似，记录以加载的系统内核模块和内核检测到的硬件设备信息，置于内存不占硬盘位置
├── tmp   # 暂存目录
├── usr   # 可分享的不可变动数据，包含大多数用户的数据和应用程序，即是软件装这里
└── var   # 动态变动文件目录，记录系统的缓存cache状态、log等数据，最好独立安装在一个分区里

21 directories, 1 file
```
### /bin
放系统自带的系统运行必要命令，单用户模式下还能被使用
```
[sunnylinux@centOSlearning bin]$ ls|grep chmod
chmod
[sunnylinux@centOSlearning bin]$ ls|grep chown
chown
```
此处和 /usr/bin 做了链接
### /boot
grub 放置引导装置程序</br>
vmlinuz 开头为内核常用文件</br>
```
[sunnylinux@centOSlearning /]$ cd boot
[sunnylinux@centOSlearning boot]$ ls
config-3.10.0-693.2.2.el7.centos.plus.i686               symvers-3.10.0-693.2.2.el7.centos.plus.i686.gz
config-3.10.0-862.2.3.el7.centos.plus.i686               symvers-3.10.0-862.2.3.el7.centos.plus.i686.gz
efi                                                      System.map-3.10.0-693.2.2.el7.centos.plus.i686
grub                                                     System.map-3.10.0-862.2.3.el7.centos.plus.i686
grub2                                                    vmlinuz-0-rescue-c186467137644f039846081199c8b99d
initramfs-0-rescue-c186467137644f039846081199c8b99d.img  vmlinuz-3.10.0-693.2.2.el7.centos.plus.i686
initramfs-3.10.0-693.2.2.el7.centos.plus.i686.img        vmlinuz-3.10.0-862.2.3.el7.centos.plus.i686
initramfs-3.10.0-862.2.3.el7.centos.plus.i686.img        xfsdumpl1test.img
```
### /dev
设备挂载点
```
/dev/null
/dev/zero
/dev/tty
/dev/lp*
/dev/hd*
/dev/sd*
```
```
[sunnylinux@centOSlearning /]$ cd /dev
[sunnylinux@centOSlearning dev]$ ls
agpgart          disk     hugepages     midi                raw       stderr  tty18  tty30  tty43  tty56  ttyS2    vcs6
autofs           dm-0     hwrng         mqueue              rtc       stdin   tty19  tty31  tty44  tty57  ttyS3    vcsa
block            dm-1     initctl       net                 rtc0      stdout  tty2   tty32  tty45  tty58  uhid     vcsa1
bsg              dm-2     input         network_latency     sda       tty     tty20  tty33  tty46  tty59  uinput   vcsa2
btrfs-control    dmmidi   kmsg          network_throughput  sda1      tty0    tty21  tty34  tty47  tty6   urandom  vcsa3
bus              dri      log           null                sda2      tty1    tty22  tty35  tty48  tty60  usbmon0  vcsa4
cdrom            fb0      loop-control  nvram               sda3      tty10   tty23  tty36  tty49  tty61  usbmon1  vcsa5
centos           fd       lp0           oldmem              sda4      tty11   tty24  tty37  tty5   tty62  usbmon2  vcsa6
char             fd0      lp1           parport0            sg0       tty12   tty25  tty38  tty50  tty63  vcs      vfio
console          full     lp2           port                sg1       tty13   tty26  tty39  tty51  tty7   vcs1     vga_arbiter
core             fuse     lp3           ppp                 shm       tty14   tty27  tty4   tty52  tty8   vcs2     vhci
cpu              hidraw0  mapper        ptmx                snapshot  tty15   tty28  tty40  tty53  tty9   vcs3     vhost-net
cpu_dma_latency  hidraw1  mcelog        pts                 snd       tty16   tty29  tty41  tty54  ttyS0  vcs4     vmci
crash            hpet     mem           random              sr0       tty17   tty3   tty42  tty55  ttyS1  vcs5     zero
```
### /etc
/etc 放置各种系统配置文件，不可分享，仅和自身系统有关，比如用户账号密码文件、各种服务起始文件等。</br>

#### 启动脚本目录 /etc/init.d
用 yum 安装的软件的配置文件会被放置在 /etc 当中，可使用各种启动脚本，启动脚本会放在/etc/init.d中
```
# 系统服务默认启动脚本
[sunnylinux@centOSlearning etc]$ ls /etc/init.d
functions  netconsole  network  README  vmware-tools  vmware-tools-thinprint
```
其他服务启动脚本:</br>
源码编译安装貌似不能用 service 相关的命令
```
# nginx 相关
sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx stop
sudo /etc/init.d/nginx restart

sudo service nginx start
sudo service nginx stop
sudo service nginx restart
sudo service nginx status

# 防火墙相关
sudo /etc/init.d/iptable start
sudo /etc/init.d/iptable stop
sudo /etc/init.d/iptable restart
```
有一些可能不会放置在/etc/init.d中，但一般会在/etc中，具体查软件的manual
```
# php 相关启动目录和配置文件
[sunnylinux@centOSlearning etc]$ ls |grep php
php.d
php-fpm.conf
php-fpm.d
php.ini
```
#### /etc/sysconfig/* 初始化环境配置文件
```
[sunnylinux@centOSlearning etc]$ ls ./sysconfig
anaconda         htcacheclean      network-scripts  run-parts
atd              httpd             ntpd             samba
authconfig       init              ntpdate          selinux
cbq              ip6tables-config  php-fpm          smartmontools
chronyd          iptables-config   qemu-ga          sshd
console          irqbalance        raid-check       svnserve
cpupower         kdump             rdisc            sysstat
crond            kernel            readonly-root    sysstat.ioconf
ebtables-config  man-db            rpcbind          wpa_supplicant
fcoe             modules           rpc-rquotad
firewalld        netconsole        rsyncd
grub             network           rsyslog
```
日志文件初始化设置
```
[sunnylinux@centOSlearning etc]$ ls ./sysconfig/rsyslog
./sysconfig/rsyslog
[sunnylinux@centOSlearning etc]$ cat ./sysconfig/rsyslog
# Options for rsyslogd
# Syslogd options are deprecated since rsyslog v3.
# If you want to use them, switch to compatibility mode 2 by "-c 2"
# See rsyslogd(8) for more details
SYSLOGD_OPTIONS=""
```
网络设置
```
[sunnylinux@centOSlearning etc]$ cat ./sysconfig/network
# Created by anaconda
```
#### daemon 配置文件目录
X window 的配置文件
```
[sunnylinux@centOSlearning etc]$ ls |grep X11
X11
```
crontab 相关的配置文件
```
[sunnylinux@centOSlearning etc]$ ls |grep cron
anacrontab
cron.d
cron.daily
cron.deny
cron.hourly
cron.monthly
crontab
cron.weekly
```
super daemon 管理的各项服务的配置文件目录</br>
super daemon 是网络请求过来了才启动的daemon，请求结束了会自动关闭释放资源，比如 telnet。super daemon 通过一个统一的daemon来管理，这个daemon是xinetd，xinetd 自己的配置文件是/etc/xinetd.conf，xinetd 所管理的各个daemon 的配置文件是 /etc/xinetd.d/\*</br>
```
[sunnylinux@centOSlearning etc]$ ls |grep xinetd.d
xinetd.d
```
注意和/etc其他各个配置文件的区别， /etc/xinetd.d/* 主要是管理机制。</br>
#### 运行级别目录
```
[sunnylinux@centOSlearning etc]$ ls | grep rc*
grep: rc1.d: 是一个目录
grep: rc2.d: 是一个目录
grep: rc3.d: 是一个目录
grep: rc4.d: 是一个目录
grep: rc5.d: 是一个目录
grep: rc6.d: 是一个目录
grep: rc.d: 是一个目录
```


### /home
```
[sunnylinux@centOSlearning /]$ ls /home
mysql  sunnylinux  sunnylinux2  www
```
### /lib
```
# 内核相关模块
[sunnylinux@centOSlearning /]$ ls /lib|grep modules
modules
modules-load.d
speech-dispatcher-modules
```
### /proc
```
[sunnylinux@centOSlearning /]$ ls /proc
1     1698  2186  2465  2579  264   2740  33   382   551   580  665   acpi         interrupts  mounts         timer_list
10    17    2193  2467  2581  2650  2742  348  383   552   602  6733  asound       iomem       mpt            timer_stats
11    1701  22    2477  2582  266   2744  349  384   553   604  676   buddyinfo    ioports     mtrr           tty
1181  18    2244  2478  2599  267   2750  35   39    554   606  7     bus          irq         net            uptime
1203  1829  2248  2482  260   2680  2794  359  454   556   628  7338  cgroups      kallsyms    pagetypeinfo   version
1205  1892  2253  2496  2600  2687  2834  36   4749  558   629  737   cmdline      kcore       partitions     vmallocinfo
1206  1897  23    25    2601  2688  2907  360  4750  559   636  8     consoles     keys        sched_debug    vmstat
1239  19    2338  251   2608  269   2962  37   479   5622  637  816   cpuinfo      key-users   schedstat      zoneinfo
1240  1995  2356  2545  261   270   3     373  489   5626  642  8229  crypto       kmsg        scsi
1241  2     2361  2550  2612  2708  3027  374  4909  5631  643  838   devices      kpagecount  self
1258  20    2366  2555  2614  2717  3047  375  4917  573   644  84    diskstats    kpageflags  slabinfo
13    2147  2395  2561  2617  272   3246  376  4919  574   645  8503  dma          loadavg     softirqs
1352  2158  24    2570  2618  2721  3252  377  5     575   646  857   driver       locks       stat
14    2162  2406  2571  2620  2723  3255  378  52    576   648  9     execdomains  mdstat      swaps
15    2172  2441  2572  2621  2727  3271  379  548   577   650  9007  fb           meminfo     sys
16    2173  2443  2574  2622  273   3276  380  549   578   657  9036  filesystems  misc        sysrq-trigger
1647  2174  2457  2576  263   2732  3296  381  550   579   662  9207  fs           modules     sysvipc
```
查询各种信息
```
# cpu 信息
[sunnylinux@centOSlearning /]$ cat /proc/cpuinfo
processor       : 0
vendor_id       : AuthenticAMD
cpu family      : 16
model           : 6
model name      : AMD Turion(tm) II Dual-Core Mobile M520
stepping        : 2
microcode       : 0x1000098
cpu MHz         : 2294.300
cache size      : 512 KB
physical id     : 0
siblings        : 1
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
fdiv_bug        : no
f00f_bug        : no
coma_bug        : no
fpu             : yes
fpu_exception   : yes
cpuid level     : 5
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm 3dnowext 3dnow constant_tsc art tsc_reliable nonstop_tsc pni cx16 x2apic popcnt hypervisor lahf_lm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw retpoline_amd ibp_disable vmmcall
bogomips        : 4588.60
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:

# 内存信息
$ cat /proc/vmstat  

$ cat /proc/dma
 2: floppy
 4: cascade
```
```
/proc/buddyinfo 每个内存区中的每个order有多少块可用，和内存碎片问题有关

/proc/cmdline 启动时传递给kernel的参数信息

/proc/cpuinfo cpu的信息

/proc/crypto 内核使用的所有已安装的加密密码及细节

/proc/devices 已经加载的设备并分类

/proc/dma 已注册使用的ISA DMA频道列表

/proc/execdomains Linux内核当前支持的execution domains

/proc/fb 帧缓冲设备列表，包括数量和控制它的驱动

/proc/filesystems 内核当前支持的文件系统类型

/proc/interrupts x86架构中的每个IRQ中断数

/proc/iomem 每个物理设备当前在系统内存中的映射

/proc/ioports 一个设备的输入输出所使用的注册端口范围

/proc/kcore 代表系统的物理内存，存储为核心文件格式，里边显示的是字节数，等于RAM大小加上4kb

/proc/kmsg 记录内核生成的信息，可以通过/sbin/klogd或/bin/dmesg来处理

/proc/loadavg 根据过去一段时间内CPU和IO的状态得出的负载状态，与uptime命令有关

/proc/locks 内核锁住的文件列表

/proc/mdstat 多硬盘，RAID配置信息(md=multiple disks)

/proc/meminfo RAM使用的相关信息

/proc/misc 其他的主要设备(设备号为10)上注册的驱动

/proc/modules 所有加载到内核的模块列表

/proc/mounts 系统中使用的所有挂载

/proc/mtrr 系统使用的Memory Type Range Registers (MTRRs)

/proc/partitions 分区中的块分配信息

/proc/pci 系统中的PCI设备列表

/proc/slabinfo 系统中所有活动的 slab 缓存信息

/proc/stat 所有的CPU活动信息

/proc/sysrq-trigger 使用echo命令来写这个文件的时候，远程root用户可以执行大多数的系统请求关键命令，就好像在本地终端执行一样。要写入这个文件，需要把/proc/sys/kernel/sysrq不能设置为0。这个文件对root也是不可读的

/proc/uptime 系统已经运行了多久

/proc/swaps 交换空间的使用情况

/proc/version Linux内核版本和gcc版本

/proc/bus 系统总线(Bus)信息，例如pci/usb等

/proc/driver 驱动信息

/proc/fs 文件系统信息

/proc/ide ide设备信息

/proc/irq 中断请求设备信息

/proc/net 网卡设备信息

/proc/scsi scsi设备信息

/proc/tty tty设备信息

/proc/net/dev 显示网络适配器及统计信息

/proc/vmstat 虚拟内存统计信息

/proc/vmcore 内核panic时的内存映像

/proc/diskstats 取得磁盘信息

/proc/schedstat kernel调度器的统计信息

/proc/zoneinfo 显示内存空间的统计信息，对分析虚拟内存行为很有用

以下是/proc目录中进程N的信息

/proc/N pid为N的进程信息

/proc/N/cmdline 进程启动命令

/proc/N/cwd 链接到进程当前工作目录

/proc/N/environ 进程环境变量列表

/proc/N/exe 链接到进程的执行命令文件

/proc/N/fd 包含进程相关的所有的文件描述符

/proc/N/maps 与进程相关的内存映射信息

/proc/N/mem 指代进程持有的内存，不可读

/proc/N/root 链接到进程的根目录

/proc/N/stat 进程的状态

/proc/N/statm 进程使用的内存的状态

/proc/N/status 进程状态信息，比stat/statm更具可读性

/proc/self 链接到当前正在运行的进程
```
### /usr
unix software resource 与软件安装/执行有关
```
[sunnylinux@centOSlearning usr]$ tree -L 1
.
├── bin  # 放置用户自己安装的命令
├── include  # 源码安装时，C/C++ 放置头文件和包含文件的地方
├── lib  # 放置自安装软件的函数库、目标文件、不常被使用的script
├── libexec  
├── local  # 源码安装软件常用目录
├── sbin   # 非系统正常运行需要的命令
├── share  # 放置共享文件的地方
├── src    # 一般源码建议放这里，内核源码放/usr/src/linux下
└── tmp -> ../var/tmp

11 directories, 0 files

```
#### /usr/bin
可以看做自己安装的命令放这里
```
[sunnylinux@centOSlearning bin]$ ls| grep python
abrt-action-analyze-python
python
python2
python2.7
python3

[sunnylinux@centOSlearning bin]$ ls| grep tree
gconf-merge-tree
gvfs-tree
nl-tctree-list
pstree
pstree.x11
tree
verifytree

[sunnylinux@centOSlearning bin]$ ls| grep virtualenv
virtualenv

[sunnylinux@centOSlearning bin]$ ls| grep glances
glances
```
#### /usr/local
```
[sunnylinux@centOSlearning local]$ tree -L 2
.
├── bin
│   ├── pcre-config
│   ├── pcregrep
│   └── pcretest
├── etc
├── games
├── include
│   ├── pcrecpparg.h
│   ├── pcrecpp.h
│   ├── pcre.h
│   ├── pcreposix.h
│   ├── pcre_scanner.h
│   └── pcre_stringpiece.h
├── lib
│   ├── libpcre.a
│   ├── libpcrecpp.a
│   ├── libpcrecpp.la
│   ├── libpcrecpp.so -> libpcrecpp.so.0.0.0
│   ├── libpcrecpp.so.0 -> libpcrecpp.so.0.0.0
│   ├── libpcrecpp.so.0.0.0
│   ├── libpcre.la
│   ├── libpcreposix.a
│   ├── libpcreposix.la
│   ├── libpcreposix.so -> libpcreposix.so.0.0.2
│   ├── libpcreposix.so.0 -> libpcreposix.so.0.0.2
│   ├── libpcreposix.so.0.0.2
│   ├── libpcre.so -> libpcre.so.1.2.3
│   ├── libpcre.so.1 -> libpcre.so.1.2.3
│   ├── libpcre.so.1.2.3
│   └── pkgconfig
├── libexec
├── mongodb-linux-i686-3.2.20
│   ├── bin
│   ├── data
│   ├── GNU-AGPL-3.0
│   ├── MPL-2
│   ├── README
│   └── THIRD-PARTY-NOTICES
├── mysql
│   ├── data
│   └── mysql-8.0.12-linux-glibc2.12-i686
├── mysql-8.0.12-linux-glibc2.12-i686
│   ├── bin
│   ├── docs
│   ├── include
│   ├── lib
│   ├── LICENSE
│   ├── man
│   ├── README
│   ├── share
│   └── support-files
├── pcre
│   ├── bin
│   ├── include
│   ├── lib
│   └── share
├── python3
│   ├── bin
│   ├── ez_setup.py
│   ├── get-pip.py
│   ├── include
│   ├── lib
│   ├── pip-18.0
│   ├── pip-18.0.tar.gz
│   ├── Python-3.7.0a1
│   ├── setuptools-19.6.tar.gz
│   ├── setuptools-40.0.0
│   ├── setuptools-40.0.0.zip
│   └── share
├── sbin
├── share
│   ├── applications
│   ├── doc
│   ├── info
│   └── man
├── src
│   ├── nginx-1.6.2
│   ├── node_modules
│   ├── node-v8.11.3-linux-x86
│   └── pcre-8.35
└── webserver
    └── nginx

47 directories, 35 files

```
### /var
与系统运行过程相关
```
[sunnylinux@centOSlearning var]$ tree -L 1
.
├── account  
├── adm
├── cache  # 应用程序运行时产生的暂存文件
├── crash
├── db
├── empty
├── gopher
├── kerberos
├── lib   # 程序执行过程中需要的数据放置地点
├── local
├── lock -> ../run/lock  # 确保某设备或文件资源仅被单个程序使用
├── log  # 各种日志、登录文件放置目录
├── mail -> spool/mail
├── nis
├── opt
├── preserve
├── run -> ../run  # 放置进程PID的地方
├── spool  # 放置队列数据，即等待被其他程序使用的数据
├── tmp
├── www
└── yp

22 directories, 0 files
```
#### /var/log
```
[sunnylinux@centOSlearning lib]$ cd /var/log
[sunnylinux@centOSlearning log]$ ls
anaconda           cups                messages-20190127  spooler-20190120
audit              dmesg               messages-20190204  spooler-20190127
boot.log           dmesg.old           ntpstats           spooler-20190204
boot.log-20190122  firewalld           php-fpm            tallylog
boot.log-20190127  gdm                 pluto              tuned
boot.log-20190129  grubby              ppp                wpa_supplicant.log
boot.log-20190130  grubby_prune_debug  qemu-ga            wtmp
boot.log-20190202  httpd               rhsm               wtmp1
boot.log-20190204  lastlog             sa                 wtmp2
boot.log-20190205  maillog             samba              Xorg.0.log
btmp               maillog-20190113    secure             Xorg.0.log.old
btmp-20190201      maillog-20190120    secure-20190113    Xorg.1.log
chrony             maillog-20190127    secure-20190120    Xorg.1.log.old
cron               maillog-20190204    secure-20190127    Xorg.2.log
cron-20190113      mariadb             secure-20190204    Xorg.2.log.old
cron-20190120      messages            speech-dispatcher  Xorg.9.log
cron-20190127      messages-20190113   spooler            yum.log
cron-20190204      messages-20190120   spooler-20190113   yum.log-20190105
```
查看cron的运行日志
```
[sunnylinux@centOSlearning log]$ sudo tail -f -n 5 /var/log/cron
Feb  6 13:10:01 centOSlearning CROND[12996]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb  6 19:10:01 centOSlearning CROND[13444]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb  6 19:20:02 centOSlearning CROND[14023]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb  6 19:30:02 centOSlearning CROND[14644]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb  6 19:40:02 centOSlearning CROND[15144]: (root) CMD (/usr/lib/sa/sa1 1 1)
```
#### /var/run
script可在这里读取某些程序当前运行的进程pid
```
[sunnylinux@centOSlearning log]$ cd /run
[sunnylinux@centOSlearning run]$ ls
abrt             dmeventd-server  lvmetad.pid     samba           tuned
alsactl.pid      ebtables.lock    mariadb         sepermit        udev
atd.pid          faillock         mdadm           setrans         udisks2
auditd.pid       firewalld        mount           setroubleshoot  user
avahi-daemon     gdm              netreport       spice-vdagentd  utmp
console          httpd            NetworkManager  sshd.pid        vmblock-fuse
crond.pid        initramfs        php-fpm         sudo            vmtoolsd.pid
cron.reboot      lock             plymouth        svnserve        xtables.lock
cups             log              ppp             syslogd.pid
dbus             lsm              rpcbind         systemd
dmeventd-client  lvm              rpcbind.sock    tmpfiles.d
[sunnylinux@centOSlearning run]$ cat crond.pid
1240
[sunnylinux@centOSlearning run]$ ps aux|grep crond
root      1240  0.0  0.0   7980   568 ?        Ss   10:04   0:02 /usr/sbin/cron  -n
sunnyli+ 15453  0.0  0.0   6708   884 pts/2    S+   19:45   0:00 grep --color=auto crond
```
# /var/spool
```
[sunnylinux@centOSlearning spool]$ ls
abrt  abrt-upload  anacron  at  cron  cups  lpd  mail  plymouth  postfix
[sunnylinux@centOSlearning spool]$ tree -L 2
.
├── abrt [error opening dir]
├── abrt-upload [error opening dir]
├── anacron
│   ├── cron.daily
│   ├── cron.monthly
│   └── cron.weekly
├── at [error opening dir]
├── cron [error opening dir]
├── cups [error opening dir]
├── lpd
├── mail  # 用户的邮箱
│   ├── mysql
│   ├── root
│   ├── rpc
│   ├── sunnylinux
│   ├── sunnylinux2
│   └── www
├── plymouth
└── postfix
    ├── active
    ├── bounce
    ├── corrupt
    ├── defer
    ├── deferred
    ├── flush
    ├── hold
    ├── incoming
    ├── maildrop
    ├── pid
    ├── private
    ├── public
    ├── saved
    └── trace

24 directories, 9 files

```
查看邮箱
```
[sunnylinux@centOSlearning spool]$ cat /var/spool/mail/sunnylinux  # 可能会有大堆报错信息 
```
