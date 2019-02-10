# Linux 日志系统
## 关于日志
日志类型
* 系统日志
* 应用日志

日志收集方式
* 由软件开发商自己来自定义日志格式然后指定输出日志位置，如 /usr/local/nginx/log 中的 log
* Linux 提供的日志服务程序，而我们这里系统日志是通过 rsyslog 来实现，提供日志管理服务，如 /var/log 中的 log

## 常见日志
ubuntu 常见log
```
日志名称	记录信息
alternatives.log	系统的一些更新替代信息记录
apport.log	应用程序崩溃信息记录
apt/history.log	使用 apt-get 安装卸载软件的信息记录
apt/term.log	使用 apt-get 时的具体操作，如 package 的下载、打开等
auth.log	登录认证的信息记录
boot.log	系统启动时的程序服务的日志信息
btmp	错误的信息记录
Consolekit/history	控制台的信息记录
dist-upgrade	dist-upgrade 这种更新方式的信息记录
dmesg	启动时，显示屏幕上内核缓冲信息,与硬件有关的信息
dpkg.log	dpkg 命令管理包的日志。
faillog	用户登录失败详细信息记录
fontconfig.log	与字体配置有关的信息记录
kern.log	内核产生的信息记录，在自己修改内核时有很大帮助
lastlog	用户的最近信息记录
wtmp	登录信息的记录。wtmp可以找出谁正在进入系统，谁使用命令显示这个文件或信息等
syslog	系统信息记录
```
CentOS7 常见log
```
[sunnylinux@centOSlearning log]$ pwd
/var/log
[sunnylinux@centOSlearning log]$ ls
anaconda           cron-20190127       maillog-20190204   samba               wtmp
audit              cron-20190204       maillog-20190210   secure              wtmp1
boot.log           cron-20190210       mariadb            secure-20190120     wtmp2
boot.log-20190127  cups                messages           secure-20190127     Xorg.0.log
boot.log-20190129  dmesg               messages-20190120  secure-20190204     Xorg.0.log.old
boot.log-20190130  dmesg.old           messages-20190127  secure-20190210     Xorg.1.log
boot.log-20190202  firewalld           messages-20190204  speech-dispatcher   Xorg.1.log.old
boot.log-20190204  gdm                 messages-20190210  spooler             Xorg.2.log
boot.log-20190205  grubby              ntpstats           spooler-20190120    Xorg.2.log.old
boot.log-20190208  grubby_prune_debug  php-fpm            spooler-20190127    Xorg.9.log
btmp               httpd               pluto              spooler-20190204    yum.log
btmp-20190201      lastlog             ppp                spooler-20190210    yum.log-20190105
chrony             maillog             qemu-ga            tallylog
cron               maillog-20190120    rhsm               tuned
cron-20190120      maillog-20190127    sa                 wpa_supplicant.log
```
/var/log/boot.log 存储本次开机资讯</br>
可以按日期细分，细分了之后 boot.log 是空的
```
[sunnylinux@centOSlearning log]$ sudo cat /var/log/boot.log
[sunnylinux@centOSlearning log]$ sudo cat /var/log/boot.log-20190208
[  OK  ] Started Show Plymouth Boot Screen.
[  OK  ] Reached target Paths.
[  OK  ] Reached target Basic System.
[  OK  ] Found device /dev/mapper/centos-root.
         Starting File System Check on /dev/mapper/centos-root...
[  OK  ] Started File System Check on /dev/mapper/centos-root.
[  OK  ] Started dracut initqueue hook.
[  OK  ] Reached target Remote File Systems (Pre).
```
/var/log/cron 定时任务日志
```
[sunnylinux@centOSlearning log]$ sudo tail -f -n 5 /var/log/cron
Feb 10 16:10:01 centOSlearning CROND[9124]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb 10 16:20:01 centOSlearning CROND[9139]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb 10 16:30:02 centOSlearning CROND[9153]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb 10 16:40:01 centOSlearning CROND[9167]: (root) CMD (/usr/lib/sa/sa1 1 1)
Feb 10 16:50:01 centOSlearning CROND[9202]: (root) CMD (/usr/lib/sa/sa1 1 1)
```
/var/log/dmesg 硬件侦测过程
```
[sunnylinux@centOSlearning log]$ sudo cat /var/log/dmesg|grep XFS
[    4.383244] SGI XFS with ACLs, security attributes, no debug enabled
[    4.394124] XFS (dm-0): Mounting V5 Filesystem
[    4.970143] XFS (dm-0): Starting recovery (logdev: internal)
[    5.005434] XFS (dm-0): Ending recovery (logdev: internal)
[   20.319728] XFS (sda2): Mounting V5 Filesystem
[   20.627794] XFS (sda2): Starting recovery (logdev: internal)
[   20.747227] XFS (sda2): Ending recovery (logdev: internal)
[   21.226330] XFS (dm-2): Mounting V5 Filesystem
[   22.014868] XFS (dm-2): Starting recovery (logdev: internal)
[   22.157416] XFS (dm-2): Ending recovery (logdev: internal)
```
/var/log/lastlog 最近登录日志</br>
/var/log/wtmp 记录登录行为，常用于追踪一般用户的登录行为</br>
lastlog 和 wtmp  并不是 ASCII 文件而是被编码成了二进制文件，所以我们并不能直接使用 less、cat、more 这样的工具来查看</br>
使用 last 与 lastlog 工具来提取其中的信息</br>
```
[sunnylinux@centOSlearning log]$ last |grep Feb
sunnylin tty1                          Sun Feb 10 14:40 - 14:41  (00:00)
sunnylin tty4                          Sun Feb 10 14:38   still logged in
sunnylin pts/0        tommy-pc.mshome. Sat Feb  9 06:44   still logged in
sunnylin pts/1        tommy-pc.mshome. Fri Feb  8 14:57 - 00:02  (09:04)
sunnylin pts/0        tommy-pc.mshome. Fri Feb  8 09:57 - 15:47  (05:50)
reboot   system boot  3.10.0-862.2.3.e Fri Feb  8 09:41 - 17:04 (2+07:22)
sunnylin pts/1        tommy-pc.mshome. Wed Feb  6 22:23 - crash (1+11:18)
sunnylin pts/2        tommy-pc.mshome. Wed Feb  6 19:10 - 04:44  (09:34)
sunnylin pts/1        tommy-pc.mshome. Wed Feb  6 10:47 - 20:49  (10:01)
sunnylin pts/1        tommy-pc.mshome. Wed Feb  6 00:05 - 01:21  (01:16)
sunnylin pts/0        :0               Wed Feb  6 00:04 - crash (2+09:37)
sunnylin pts/1        tommy-pc.mshome. Tue Feb  5 23:02 - 00:03  (01:00)
sunnylin :0           :0               Tue Feb  5 15:15 - crash (2+18:26)
sunnylin pts/0        tommy-pc.mshome. Tue Feb  5 15:13 - 23:14  (08:00)
reboot   system boot  3.10.0-862.2.3.e Tue Feb  5 15:07 - 17:04 (5+01:56)
sunnylin :0           :0               Mon Feb  4 10:09 - crash (1+04:58)
sunnylin pts/0        tommy-pc.mshome. Mon Feb  4 08:45 - crash (1+06:21)
reboot   system boot  3.10.0-862.2.3.e Mon Feb  4 08:41 - 17:04 (6+08:22)
sunnylin pts/0        tommy-pc.mshome. Sat Feb  2 10:22 - 14:49  (04:26)
sunnylin pts/2        tommy-pc.mshome. Sat Feb  2 00:17 - 11:48  (11:30)
sunnylin pts/3        :0               Fri Feb  1 09:23 - 09:23  (00:00)
sunnylin pts/2        :0               Fri Feb  1 09:23 - 09:23  (00:00)
sunnylin pts/1        :0               Fri Feb  1 09:23 - 14:50 (1+05:26)
sunnylin :0           :0               Fri Feb  1 09:21 - crash (2+23:20)
sunnylin pts/0        tommy-pc.mshome. Fri Feb  1 08:45 - 01:10  (16:24)
reboot   system boot  3.10.0-862.2.3.e Fri Feb  1 08:35 - 17:04 (9+08:28)
sunnylin pts/7        :0               Fri Feb  1 06:49 - 06:49  (00:00)
sunnylin pts/6        :0               Fri Feb  1 06:49 - 06:49  (00:00)
sunnylin pts/5        :0               Fri Feb  1 06:49 - 06:49  (00:00)
sunnylin pts/4        :0               Fri Feb  1 06:49 - 06:49  (00:00)
sunnylin pts/3        :0               Fri Feb  1 06:49 - 06:49  (00:00)
sunnylin pts/1        :0               Fri Feb  1 06:49 - 08:28  (01:38)
sunnylin pts/2        tommy-pc.mshome. Fri Feb  1 03:45 - 08:26  (04:41)

[sunnylinux@centOSlearning log]$ lastlog --help
用法：lastlog [选项]

选项：
  -b, --before DAYS             仅打印早于 DAYS 的最近登录记录
  -C, --clear                   clear lastlog record of an user (usable only with -u)
  -h, --help                    显示此帮助信息并推出
  -R, --root CHROOT_DIR         chroot 到的目录
  -S, --set                     set lastlog record to current time (usable only with -u)
  -t, --time DAYS               仅打印晚于 DAYS 的最近登录记录
  -u, --user LOGIN              打印 LOGIN 用户的最近登录记录

[sunnylinux@centOSlearning ~]$ lastlog -u root
用户名           端口     来自             最后登陆时间
root             pts/1                     六 2月  2 10:42:17 +0800 2019
[sunnylinux@centOSlearning ~]$ lastlog -u sunnylinux
用户名           端口     来自             最后登陆时间
sunnylinux       pts/0    tommy-pc.mshome. 日 2月 10 17:08:14 +0800 2019
```
/var/log/messages 记录系统的各种错误信息
```
[sunnylinux@centOSlearning ~]$ sudo tail -n 20 /var/log/messages
[sudo] sunnylinux 的密码：
Feb 10 17:03:41 centOSlearning systemd: Started Fingerprint Authentication Daemon.
Feb 10 17:03:41 centOSlearning journal: D-Bus service launched with name: net.reactivated.Fprint
Feb 10 17:03:41 centOSlearning fprintd: Launching FprintObject
Feb 10 17:03:41 centOSlearning journal: entering main loop
Feb 10 17:04:11 centOSlearning journal: No devices in use, exit
Feb 10 17:07:52 centOSlearning systemd-logind: Removed session 102.
Feb 10 17:08:14 centOSlearning systemd: Started Session 152 of user sunnylinux.
Feb 10 17:08:14 centOSlearning systemd-logind: New session 152 of user sunnylinux.
Feb 10 17:08:14 centOSlearning systemd: Starting Session 152 of user sunnylinux.
Feb 10 17:08:14 centOSlearning dbus[638]: [system] Activating service name='org.freedesktop.problems' (using servicehelper)
Feb 10 17:08:14 centOSlearning dbus[638]: [system] Successfully activated service 'org.freedesktop.problems'
Feb 10 17:10:01 centOSlearning systemd: Started Session 153 of user root.
Feb 10 17:10:01 centOSlearning systemd: Starting Session 153 of user root.
Feb 10 17:11:24 centOSlearning dbus[638]: [system] Activating via systemd: service name='net.reactivated.Fprint' unit='fprintd.service'
Feb 10 17:11:24 centOSlearning systemd: Starting Fingerprint Authentication Daemon...
Feb 10 17:11:24 centOSlearning dbus[638]: [system] Successfully activated service 'net.reactivated.Fprint'
Feb 10 17:11:24 centOSlearning systemd: Started Fingerprint Authentication Daemon.
Feb 10 17:11:24 centOSlearning journal: D-Bus service launched with name: net.reactivated.Fprint
Feb 10 17:11:24 centOSlearning fprintd: Launching FprintObject
Feb 10 17:11:24 centOSlearning journal: entering main loop
```
/var/log/secure 记录需要输密码的行为，不管是否失败，仅记录开机之后的数据
```
[sunnylinux@centOSlearning log]$ sudo tail -n 5 /var/log/secure
Feb 10 17:11:28 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/tail -n 20 /var/log/messages
Feb 10 17:21:53 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/cat /var/log/secure
Feb 10 17:22:18 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/cat /var/log/secure
Feb 10 17:22:22 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/cat /var/log/secure
Feb 10 17:22:40 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/tail -n 5 /var/log/secure

[sunnylinux@centOSlearning log]$ sudo cat /var/log/secure-20190204|grep php
Jan 28 00:27:52 centOSlearning sudo: sunnylinux : TTY=pts/0 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/yum install php5-fpm
Jan 31 06:29:14 centOSlearning sudo: sunnylinux : TTY=pts/2 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/yum intall php
Jan 31 06:29:22 centOSlearning sudo: sunnylinux : TTY=pts/2 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/yum install php
Jan 31 06:35:49 centOSlearning sudo: sunnylinux : TTY=pts/2 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/sbin/service php start
Jan 31 06:36:36 centOSlearning polkitd[635]: Operator of unix-process:9028:5251384 successfully authenticated as unix-user:sunnylinux to gain ONE-SHOT authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.510 [systemctl start php] (owned by unix-user:sunnylinux)
Jan 31 06:41:35 centOSlearning polkitd[635]: Operator of unix-process:9079:5281356 successfully authenticated as unix-user:sunnylinux to gain ONE-SHOT authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.518 [/bin/systemctl start php.service] (owned by unix-user:sunnylinux)
Jan 31 06:42:25 centOSlearning sudo: sunnylinux : TTY=pts/2 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/yum install php-fpm
Jan 31 06:56:46 centOSlearning polkitd[635]: Operator of unix-process:9201:5372514 successfully authenticated as unix-user:sunnylinux to gain ONE-SHOT authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.532 [/bin/systemctl start php-fpm.service] (owned by unix-user:sunnylinux)
Jan 31 07:12:24 centOSlearning polkitd[635]: Operator of unix-process:9473:5466425 successfully authenticated as unix-user:sunnylinux to gain ONE-SHOT authorization for action org.freedesktop.systemd1.manage-units for system-bus-name::1.559 [/bin/systemctl start php-fpm.service] (owned by unix-user:sunnylinux)
Jan 31 07:16:45 centOSlearning sudo: sunnylinux : TTY=pts/1 ; PWD=/usr/local/webserver/nginx/html ; USER=root ; COMMAND=/bin/vim phpinfo.php
Jan 31 08:09:43 centOSlearning sudo: sunnylinux : TTY=pts/1 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/bin/yum install php-mysql
Jan 31 08:16:49 centOSlearning sudo: sunnylinux : TTY=pts/1 ; PWD=/home/sunnylinux ; USER=root ; COMMAND=/sbin/service php-fpm restart
```

