# Linux daemons 启动与管理 init、systemd

## daemons 命名规则
service 名后面加 d


## daemons 启动脚本和启动方式
Daemon的启动需要执行文件、配置文件、执行环境。</br>
</br>
启动脚本:</br>
进行环境的检测、加载提供的函数功能、配置文件的分析、PID文件的放置，以及重要交换文件的锁住 lock 操作。</br>
过去，系统服务的进程大多依靠一个init的启动机制来启动，CentOS7以后使用的是 systemd。</br>

## init
### init daemons 分类
按照 daemons 的启动和管理方式分类：</br>
* stand_alone</br>
常驻于内存当中，可自启，不需要额外的管理，响应速度快。</br>
例子：httpd、vsftpd</br>
* super daemons</br>
由一个总管进程 xinetd 统一管理，有socket或port请求来时才启动，不会一直占用内存空间，启动较慢，可以具有安全控管的机制，提供IP来源、时间、连线管理等功能。</br>
例子：telnet</br>
</br>
根据工作形态分：</br>

* singal-control</br>
通过信号来管理，请求来立刻去处理。</br>
* interval-control</br>
每隔一段时间去主动执行某项工作</br>
例子：crond

### init 相关目录
init方式启动，主要涉及到/etc 和 /var 下的一些文件，[linux 目录结构](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.md#etc)。</br>
```
/etc/init.d/*    启动脚本放置处，管理一个或多个daemon的启动（以前使用）
/etc/sysconfig/*  各服务的初始化环境配置文件
/etc/xinetd.conf  super daemon 默认配置文件
/etc/xinetd.d/*   super daemon 更细致的配置文件

/var/run/  启动的daemon进程PID记录处
/var/lib/* 各服务产生的数据库
```

### init 启动
#### stand_alone daemons 启动
* /etc/init.d/* 方式</br>
CentOS7 后不用了，ubuntu 貌似还可以用
```
# 一般格式为
/etc/init.d/crond start
```
参考 [Linux(ubuntu)-Nginx-MySQL-PHP 架构 apt-get 安装配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/server_note/Nginx_note/Linux%20Nginx%20MySQL%20PHP%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.MD#%E5%B8%B8%E7%94%A8-nginx-%E5%91%BD%E4%BB%A4)
</br>

* service 方式
```
[sunnylinux@centOSlearning init.d]$ service crond
The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
```
CentOS 中可以使用 service 命令，但其实是 Redirecting to /bin/systemctl status crond.service
```
[sunnylinux@centOSlearning init.d]$ service crond status
Redirecting to /bin/systemctl status crond.service
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2019-02-08 09:44:37 CST; 2h 34min ago
 Main PID: 1221 (crond)
   CGroup: /system.slice/crond.service
           └─1221 /usr/sbin/crond -n

2月 08 09:44:37 centOSlearning.SharonLi systemd[1]: Started Command Scheduler.
2月 08 09:44:37 centOSlearning.SharonLi systemd[1]: Starting Command Scheduler...
2月 08 09:44:37 centOSlearning.SharonLi crond[1221]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 45% if used.)
2月 08 09:44:45 centOSlearning.SharonLi crond[1221]: (CRON) INFO (running with inotify support)
```

#### super daemons 
super daemons 分类（根据处理方式分）
* 单线程
* 多线程

## systemd 
systemd 特点
* 速度比 init 快，systemd 本身常驻于内存
* systemd 服务仅需使用 systemctl 命令来处理，即是不用 init, chkconfig, service 来支撑，但减少了自定义的开放性
* systemd 将服务分为一个服务单位 unit，再将unit按功能分为type，type 有 service, socket, target, path, snapshot, timer
* init 有 runlevel，而systemd 对应的是 target ，但没有全部对应，runlevel依然无可取代
* systemd 向下兼容 init
</br>

### systemd 相关目录
```
/etc/systemd/system/  可依据主机需求建立启动脚本，优先级比/run/systemd/system/ 高，功能类似于/etc/rc.d/rc5.d/Sxx，决定脚本是否开机执行
/run/systemd/system/  系统运行过程中产生的脚本，优先级比/usr/lib/systemd/system/ 高
/usr/lib/systemd/system/  启动脚本目录，类似于/etc/init.d，实际执行的启动脚本位置
```
/usr/lib/systemd/system/
```
[sunnylinux@centOSlearning system]$ pwd
/usr/lib/systemd/system
[sunnylinux@centOSlearning system]$ ls |grep cron
crond.service

# 还有很多其他脚本
tcsd.service
teamd@.service
timers.target
timers.target.wants
time-sync.target
tmp.mount
tuned.service
udisks2.service
umount.target
unbound-anchor.service
unbound-anchor.timer
upower.service
usb_modeswitch@.service
usbmuxd.service
user.slice
wacom-inputattach@.service
wpa_supplicant.service
zram.service
```
/run/systemd/system/
```
[sunnylinux@centOSlearning system]$ cd /run/systemd/system/
[sunnylinux@centOSlearning system]$ ls
session-2.scope    session-39.scope.d  user-1000.slice    user-42.slice.d
session-2.scope.d  session-c1.scope    user-1000.slice.d
session-39.scope   session-c1.scope.d  user-42.slice
```
/etc/systemd/system/
```
[sunnylinux@centOSlearning system]$ cd /etc/systemd/system/
[sunnylinux@centOSlearning system]$ ls
basic.target.wants
bluetooth.target.wants
dbus-org.bluez.service
dbus-org.fedoraproject.FirewallD1.service
dbus-org.freedesktop.Avahi.service
dbus-org.freedesktop.ModemManager1.service
dbus-org.freedesktop.NetworkManager.service
dbus-org.freedesktop.nm-dispatcher.service
default.target
default.target.wants
dev-virtio\x2dports-org.qemu.guest_agent.0.device.wants
display-manager.service
getty.target.wants
graphical.target.wants
local-fs.target.wants
multi-user.target.wants
network-online.target.wants
printer.target.wants
sockets.target.wants
spice-vdagentd.target.wants
sysinit.target.wants
system-update.target.wants
timers.target.wants
```
### systemd 服务类型
* .service   </br>
service unit  </br>
一般系统服务，包括本机服务和网络服务 </br>
* .socket    </br>
socket unit   </br>
很多软件通过socket来进行本机的数据交互，类似于 xinetd </br>
* .target	 </br>
target unit </br>
多种 unit 的集合，执行这种unit相当于执行一些列的unit</br>
* .mount .automount  
automount unit / mount unit</br>
档案系统挂载相关服务</br>
* .path    </br>
path unit</br>
某些服务需要侦测某些特定的目录来提供服务，需要 .path 来支持</br>
* .timer   </br>
timer unit</br>
类似 anacrontb 但这个是 systemd 提供的，更有弹性</br>
```
# 区分 unit的 type 类型看文件名

[sunnylinux@centOSlearning system]$ ls /usr/lib/systemd/system|grep -E '(crond|vsftp|multi)'
crond.service
multipathd.service
multi-user.target
multi-user.target.wants


[sunnylinux@centOSlearning system]$ ls /usr/lib/systemd/system|grep .socket
avahi-daemon.socket
cups.socket
dbus.socket
dm-event.socket
iscsid.socket
iscsiuio.socket
lldpad.socket
lvm2-lvmetad.socket
lvm2-lvmpolld.socket
rpcbind.socket
rsyncd.socket
spice-vdagentd.socket
sshd.socket
syslog.socket
systemd-initctl.socket
systemd-journald.socket
systemd-shutdownd.socket
systemd-udevd-control.socket
systemd-udevd-kernel.socket
```

### systemctl 启动
CentOS7 可用，systemctl 也是yum安装才能用，源码安装的用不到
```
systemctl (start|stop|restart|try-restart|reload|force-reload|status) [unit]
```
Loaded 为 enabled 则为开机自启， disabled 则不为开机自启</br>
Active 常见状态
* active (running) 正在执行
* active (exited) 现时没有正在执行的程序，多见于只执行一次就结束的服务
* active (waiting) 正在执行中，但是要等待其他服务响应
* inactive (dead) 没有在运作
```
[sunnylinux@centOSlearning init.d]$ systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2019-02-08 09:44:37 CST; 3h 22min ago
 Main PID: 1221 (crond)
   CGroup: /system.slice/crond.service
           └─1221 /usr/sbin/crond -n

2月 08 09:44:37 centOSlearning.SharonLi systemd[1]: Started Command Scheduler.
2月 08 09:44:37 centOSlearning.SharonLi systemd[1]: Starting Command Scheduler...
2月 08 09:44:37 centOSlearning.SharonLi crond[1221]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 45% if used.)
2月 08 09:44:45 centOSlearning.SharonLi crond[1221]: (CRON) INFO (running with inotify support)

[sunnylinux@centOSlearning init.d]$ systemctl restart crond
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: sunnylinux
Password:
```
注意：不应该用kill方法来关掉一些正常的服务，应该用stop
```
[sunnylinux@centOSlearning system]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2019-02-08 09:44:28 CST; 5h 55min ago
     Docs: man:firewalld(1)
 Main PID: 713 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─713 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

2月 08 09:42:11 centOSlearning.SharonLi systemd[1]: Starting firewalld - dynamic firewall daemon...
2月 08 09:44:28 centOSlearning.SharonLi systemd[1]: Started firewalld - dynamic firewall daemon.

[sunnylinux@centOSlearning system]$ systemctl stop firewalld
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===

[sunnylinux@centOSlearning system]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since 五 2019-02-08 15:40:32 CST; 3s ago
     Docs: man:firewalld(1)
  Process: 713 ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 713 (code=exited, status=0/SUCCESS)

2月 08 09:42:11 centOSlearning.SharonLi systemd[1]: Starting firewalld - dynamic firewall daemon...
2月 08 09:44:28 centOSlearning.SharonLi systemd[1]: Started firewalld - dynamic firewall daemon.
2月 08 15:40:26 centOSlearning.SharonLi systemd[1]: Stopping firewalld - dynamic firewall daemon...
2月 08 15:40:32 centOSlearning.SharonLi systemd[1]: Stopped firewalld - dynamic firewall daemon.
```
设置开机自启
```
[sunnylinux@centOSlearning system]$ systemctl status chronyd.service
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:chronyd(8)
           man:chrony.conf(5)
           
[sunnylinux@centOSlearning system]$ systemctl enable chronyd.service
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
Created symlink from /etc/systemd/system/multi-user.target.wants/chronyd.service to /usr/lib/systemd/system/chronyd.service.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===

[sunnylinux@centOSlearning system]$ systemctl status chronyd.service
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:chronyd(8)
           man:chrony.conf(5)
```
完整关闭一些服务（暂停运行+关闭开机自启）</br>
注意：要看清楚 Removed symlink 是否只有一条，如果有多条则为有相关联的服务</br>
实际上有很多服务是相互依赖的，一个服务会启动另一个服务，所以有时关了也没有用，有请求来了会自启</br>
```
[sunnylinux@centOSlearning system]$ systemctl status cups
● cups.service - CUPS Printing Service
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2019-02-08 09:44:36 CST; 6h ago
 Main PID: 1194 (cupsd)
   CGroup: /system.slice/cups.service
           └─1194 /usr/sbin/cupsd -f

2月 08 09:44:36 centOSlearning.SharonLi systemd[1]: Started CUPS Printing Service.
2月 08 09:44:36 centOSlearning.SharonLi systemd[1]: Starting CUPS Printing Service...

[sunnylinux@centOSlearning system]$ systemctl stop cups
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
Warning: Stopping cups.service, but it can still be activated by:
  cups.socket
  cups.path
  
[sunnylinux@centOSlearning system]$ systemctl disable cups
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
Removed symlink /etc/systemd/system/multi-user.target.wants/cups.path.
Removed symlink /etc/systemd/system/multi-user.target.wants/cups.service.
Removed symlink /etc/systemd/system/sockets.target.wants/cups.socket.
Removed symlink /etc/systemd/system/printer.target.wants/cups.service.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===

[sunnylinux@centOSlearning system]$ systemctl status cups
● cups.service - CUPS Printing Service
   Loaded: loaded (/usr/lib/systemd/system/cups.service; disabled; vendor preset: enabled)
   Active: inactive (dead) since 五 2019-02-08 15:58:50 CST; 39s ago
 Main PID: 1194 (code=exited, status=0/SUCCESS)

2月 08 09:44:36 centOSlearning.SharonLi systemd[1]: Started CUPS Printing Service.
2月 08 09:44:36 centOSlearning.SharonLi systemd[1]: Starting CUPS Printing Service...
2月 08 15:58:49 centOSlearning.SharonLi systemd[1]: Stopping CUPS Printing Service...
2月 08 15:58:50 centOSlearning.SharonLi systemd[1]: Stopped CUPS Printing Service.
```
cups.socket和cups.path启动后，会自动启动 CUPS.service，所以如果真的要关闭 CUPS.service 就需要把 cups.socket和cups.path 也关了。</br>
另一种方法是直接把  CUPS.service 注销掉，这样就不用关闭 cups.socket和cups.path 了，非正规但有效。
```
systemctl mask cups.service  # 注销
systemctl unmask cups.service  # 取消注销，恢复服务
```

### 查看已启动的服务
查看当前已启动的服务
```
[sunnylinux@centOSlearning system]$ systemctl
 UNIT                                             LOAD   ACTIVE SUB       DESCRIPTION
  proc-sys-fs-binfmt_misc.automount                loaded active waiting   Arbitrary Executable File Formats File System Au
  sys-devices-pci0000:00-0000:00:07.1-ata2-host2-target2:0:0-2:0:0:0-block-sr0.device loaded active plugged   VMware_Virtual_IDE_CDROM_Drive
  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda-sda1.device loaded active plugged   VMware_Virtual_S 1
  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda-sda2.device loaded active plugged   VMware_Virtual_S 2
  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda-sda3.device loaded active plugged   LVM PV NIl8Mq-jOzd-IiSN-1giH-7
  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda-sda4.device loaded active plugged   VMware_Virtual_S 4
  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda.device loaded active plugged   VMware_Virtual_S
  sys-devices-pci0000:00-0000:00:11.0-0000:02:01.0-net-ens33.device loaded active plugged   79c970 [PCnet32 LANCE] (PCnet - Fast 79C971)
  sys-devices-pci0000:00-0000:00:11.0-0000:02:02.0-sound-card0.device loaded active plugged   ES1371/ES1373 / Creative Labs CT2518 (Audio PC
  sys-devices-platform-floppy.0-block-fd0.device   loaded active plugged   /sys/devices/platform/floppy.0/block/fd0
  sys-devices-platform-serial8250-tty-ttyS2.device loaded active plugged   /sys/devices/platform/serial8250/tty/ttyS2
  sys-devices-platform-serial8250-tty-ttyS3.device loaded active plugged   /sys/devices/platform/serial8250/tty/ttyS3
  sys-devices-pnp0-00:06-tty-ttyS0.device          loaded active plugged   /sys/devices/pnp0/00:06/tty/ttyS0
  sys-devices-pnp0-00:07-tty-ttyS1.device          loaded active plugged   /sys/devices/pnp0/00:07/tty/ttyS1
  sys-devices-virtual-block-dm\x2d0.device         loaded active plugged   /sys/devices/virtual/block/dm-0
  sys-devices-virtual-block-dm\x2d1.device         loaded active plugged   /sys/devices/virtual/block/dm-1
  sys-devices-virtual-block-dm\x2d2.device         loaded active plugged   /sys/devices/virtual/block/dm-2
  sys-module-configfs.device                       loaded active plugged   /sys/module/configfs
  sys-module-fuse.device                           loaded active plugged   /sys/module/fuse
  sys-subsystem-net-devices-ens33.device           loaded active plugged   79c970 [PCnet32 LANCE] (PCnet - Fast 79C971)
  -.mount                                          loaded active mounted   /
  boot.mount                                       loaded active mounted   /boot
  dev-hugepages.mount                              loaded active mounted   Huge Pages File System
  dev-mqueue.mount                                 loaded active mounted   POSIX Message Queue File System

# 省略很多输出

LOAD   = Reflects whether the unit definition was properly loaded.  # 开机时是否被载入
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

142 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```
查看已被安装的服务</br>
static 指的是这个daemons不能开机自启，但可以被其他daemons唤醒
```
[sunnylinux@centOSlearning system]$ systemctl list-unit-files
UNIT FILE                                     STATE
proc-sys-fs-binfmt_misc.automount             static
dev-hugepages.mount                           static
dev-mqueue.mount                              static
proc-sys-fs-binfmt_misc.mount                 static
sys-fs-fuse-connections.mount                 static
sys-kernel-config.mount                       static
sys-kernel-debug.mount                        static
tmp.mount                                     disabled
brandbot.path                                 enabled
cups.path                                     disabled
systemd-ask-password-console.path             static
systemd-ask-password-plymouth.path            static
systemd-ask-password-wall.path                static
session-2.scope                               static
session-39.scope                              static
```
mask 状态表示被强制注销（非删除），无论如何都无法启动
```
[sunnylinux@centOSlearning system]$ systemctl list-unit-files |grep cups
cups.path                                     disabled
cups-browsed.service                          disabled
cups.service                                  disabled
cups.socket                                   disabled

[sunnylinux@centOSlearning system]$ systemctl mask cups.service
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
Created symlink from /etc/systemd/system/cups.service to /dev/null.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===

[sunnylinux@centOSlearning system]$ systemctl list-unit-files |grep cups
cups.path                                     disabled
cups-browsed.service                          disabled
cups.service                                  masked
cups.socket                                   disabled

[sunnylinux@centOSlearning system]$ systemctl unmask cups.service
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
Removed symlink /etc/systemd/system/cups.service.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: sunnylinux
Password:
==== AUTHENTICATION COMPLETE ===
[sunnylinux@centOSlearning system]$ systemctl list-unit-files |grep cups
cups.path                                     disabled
cups-browsed.service                          disabled
cups.service                                  disabled
cups.socket                                   disabled
```
列出类型为.service 的cpu相关服务
```
[sunnylinux@centOSlearning system]$ systemctl --type=service --all|grep cpu
  cpupower.service                                      loaded    inactive dead    Configure CPU power related settings
```
