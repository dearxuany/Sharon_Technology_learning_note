# Linux 时间同步
起因：</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/linux_time01.png) </br>
linux CentOS7 是安装在 win7 上的VMware虚拟机。</br>
今天发现win7的时间比其他机器快了3分钟，linux 时间同win7。</br>
网络同步了win7的时间后，linux的时间还是快了差不多3 mins。</br>
</br>
解决问题的方向：</br>
因为linux是虚拟机而不是真实存在的机子，所以解决问题的方向有很多。</br>
可以从VMware 入手，可以直接从linux平台上入手。</br>

## 在VMware角度上解决
### 安装Vmware tools后，把time synchronization勾上。
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/linux_time02.png) </br>
Time sync会每三分钟与宿主主机同步更新一次时间。 </br>
据说，这个只是解决虚拟机时间比物理机慢的问题，虚拟机比物理机快了是否能解决不清楚。</br>
此处不这么做，因为我觉得物理机上的时间也不可靠。</br>

### 重启虚拟机
这部虚拟机是装过Vmware tools、可以联网而且没有什么重要的任务。尝试了重启，确实可以同步虚拟机时间与物理机的时间。同步成功的根本原因是什么，还没有深究。可是虚拟机运行时间长了之后，虚拟机的时间还是会逐渐变得比物理机慢一点。


## 在linux 角度上解决
让linux连接到时间服务器并定期自动同步时间，前提是linux系统已经联网。</br>
比较推荐这种做法，因为时间服务器上的时间相对来说比较准确，直接从时间服务器上获取时间误差会比较少。</br>
### linux上的时间
* 系统时间: </br>
一般说来就是我们执行 date 命令看到的时间，linux系统下所有的时间调用（除了直接访问硬件时间的命令）都是使用的这个时间。</br>

* 硬件时间:</br>
主板上BIOS中的时间，由主板电池供电来维持运行，系统开机时要读取这个时间，并根据它来设定系统时间（注意：系统启动时根据硬件时间设定系统时间的过程可能存在时区换算，这要视具体的系统及相关设置而定）。</br>

在Linux下，默认情况下，系统时间和硬件时间，并不会自动同步。在Linux运行过程中，系统时间和硬件时间以异步的方式运行，互不干扰。硬件时间的运行，是靠Bios电池来维持，而系统时间，是用CPU tick来维持的。在系统开机的时候，硬件时钟会去读取系统时钟的设置，实现同步。如果不重启，这两个时间随着系统运行持续时间的延长会相差得越来越多。
</br>
查看硬件时间和系统时间：
```
# hwclock
2018年08月30日 星期四 20时20分39秒  -1.107251 秒
# date
2018年 08月 30日 星期四 12:20:44 CST
```
根据系统时间设置硬件时间（手动）：
```
# hwclock  -w
# hwclock
2018年08月30日 星期四 12时38分20秒  -1.046440 秒
# date
2018年 08月 30日 星期四 12:38:33 CST
```

### NTP服务器同步时间
NTP是网络时间协议(Network Time Protocol)，它是用来同步网络中各个计算机的时间的协议。它的用途是把计算机的时钟同步到世界协调时UTC，其精度在局域网内可达0.1ms，在互联网上绝大多数的地方其精度可以达到1-50ms。</br>
让linux经网络连接到ntpdate服务器并定期自动同步更新系统时间，这样就很大程度保证linux上的系统时间是准确的。注意，ntpdate服务器要找离自己主机最近的那一台，可在https://www.ntppool.org/en/ 上查找。
物理机win7上用的是 time.windows.com ，中国的是cn.pool.ntp.org 。

#### 下载安装有关ntp服务的安装包
使用yum命令安装
```
# yum install ntp ntpdate ntp-doc
```
开启服务
```
# chkconfig ntpd on
注意：正在将请求转发到“systemctl enable ntpd.service”。
Created symlink from /etc/systemd/system/multi-user.target.wants/ntpd.service to /usr/lib/systemd/system/ntpd.service.
```

#### 连接ntpd服务器并同步时间
可同时设置多个ntpd 服务器
```
# ntpdate time.windows.com cn.pool.ntp.org
30 Aug 13:23:37 ntpdate[4358]: adjust time server 51.75.17.219 offset -0.369294 sec
```

#### 设置定期同步时间间隔
这个操作是依靠cron服务的，要完成定期时间同步的任务，前提要求cron可以正常使用并能开机自启。</br>
cron是一个linux下的定时执行工具，可以在无需人工干预的情况下定时地运行任务task。</br>
详情： [Linux 例行性工作 crontab](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E4%BE%8B%E8%A1%8C%E6%80%A7%E5%B7%A5%E4%BD%9C.MD)</br>
</br>
打开配置文件/etc/crontab (用于记录需要定时执行的命令的文件)
```
# vim /etc/crontab
```
添加以下一句，让系统每两小时自动更新系统时间
```
0 */2 * * * root ntpdate time.windows.com cn.pool.ntp.org
```
重启crond
```
# service crond restart
```
至此，linux 系统时间的ntp自动同步就配置完成了。

#### 检测自动同步是否成功
查看cron 运行记录
```
# tail -f /var/log/cron
Sep  1 04:00:01 centOSlearning CROND[5251]: (root) CMD (ntpdate time.windows.com cn.pool.ntp.org)
Sep  2 16:20:02 centOSlearning CROND[5289]: (root) CMD (/usr/lib/sa/sa1 1 1)
Sep  2 16:25:59 centOSlearning crontab[5318]: (root) LIST (root)
```
可以看到ntpdate成功自动运行，而且时间自动和网络同步了。
