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
```
>>> psutil.virtual_memory()
svmem(total=1052327936, available=95428608, percent=90.9, used=785432576, free=86507520, active=359854080, inactive=368095232, buffers=0, cached=180387840, shared=5054464, slab=96137216)
```
swap分区
```
>>> psutil.swap_memory()
sswap(total=1073737728, used=15474688, free=1058263040, percent=1.4, sin=319488, sout=15335424)
```
