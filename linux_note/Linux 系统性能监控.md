# Linux 系统性能监控
Linux 系统监控一般涉及 进程、CPU、内存、硬盘使用率、硬盘IO、系统负载 等参数。

## 进程
进程相关的命令
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


## CPU
CPU相关的命令
```
lscpu
uptime
top
htop
vmstat
mpstat
```
