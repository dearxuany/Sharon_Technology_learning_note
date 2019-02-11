# Linux 日志系统
## 关于日志
日志类型
* 系统日志
* 应用日志

日志收集方式
* 由软件开发商自己来自定义日志格式然后指定输出日志位置，如 /usr/local/nginx/log 中的 log
* Linux 提供的日志服务程序，而我们这里系统日志是通过 rsyslog 来实现，提供日志管理服务，如 /var/log 中的 log

日志相关daemons</br>
* rsyslog 用于统一管理log</br>
* logrotate 用于自动化处理 log，例如分片、备份等操作（轮替）</br>
* systemd-journald systemd自己的日志管理服务，类似于rsyslog</br>
一般情况下，是systemd-journald负责收集信息，以二进制方式记录下来，然后发给rsyslog进行进一步日志管理。</br>
通过 journalctl 和 systemctl status unit.service 可查看不同服务的log，不用在 /var/log/messages 里面慢慢找。</br>


## 常见日志
不同分发版日志名不一样
### ubuntu 常见log
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
### CentOS7 常见log
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

## rsyslog 日志管理
启动 rsyslog
```
[sunnylinux@centOSlearning ~]$ systemctl status rsyslog.service
● rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since 日 2019-02-10 23:33:08 CST; 11h ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 1210 (rsyslogd)
   CGroup: /system.slice/rsyslog.service
           └─1210 /usr/sbin/rsyslogd -n

2月 10 23:33:06 centOSlearning.SharonLi systemd[1]: Starting System Logging ...
2月 10 23:33:08 centOSlearning.SharonLi rsyslogd[1210]:  [origin software="r...
2月 10 23:33:08 centOSlearning.SharonLi systemd[1]: Started System Logging S...
Hint: Some lines were ellipsized, use -l to show in full.
```
### rsyslog 配置
rsyslog 配置文件 /etc/rsyslog.conf
```
[sunnylinux@centOSlearning etc]$ ls|grep 'rsyslog'
rsyslog.conf  
rsyslog.d  # 更加细致的配置  /etc/rsyslog.d/*.conf
```
配置项：
* service name
* 等级信息
* log的位置
```
[sunnylinux@centOSlearning etc]$ sudo cat rsyslog.conf

#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
```
上面的内容对应了/var/log 中产生的log，语法结构：
```
# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!

服务名称.信息等级                                        log 存放位置
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
```
可用 man 来查看各种选项细节
```
[sunnylinux@centOSlearning etc]$ man 3 syslog

# Linux 对于服务名称有一套 syslog 的统一标准，其他软件可以通过调用这些服务名称来记录自己的log

  facility
       The facility argument is used to specify what type of program  is  log‐
       ging  the  message.  This lets the configuration file specify that mes‐
       sages from different facilities will be handled differently.

       LOG_AUTH       security/authorization messages

       LOG_AUTHPRIV   security/authorization messages (private)

       LOG_CRON       clock daemon (cron and at)

       LOG_DAEMON     system daemons without separate facility value

       LOG_FTP        ftp daemon

       LOG_KERN       kernel messages (these can't be generated from user pro‐
                      cesses)

       LOG_LOCAL0 through LOG_LOCAL7
                      reserved for local use

       LOG_LPR        line printer subsystem

       LOG_MAIL       mail subsystem

       LOG_NEWS       USENET news subsystem

       LOG_SYSLOG     messages generated internally by syslogd(8)

       LOG_USER (default)
                      generic user-level messages

       LOG_UUCP       UUCP subsystem

# 信息等级

   level
       This  determines  the  importance  of  the message.  The levels are, in
       order of decreasing importance:

       LOG_EMERG      system is unusable  # 最高等级错误，大概硬件出问题 0

       LOG_ALERT      action must be taken immediately  # 比 crit 严重的错误 1

       LOG_CRIT       critical conditions  # 比 error 严重的错误 2

       LOG_ERR        error conditions  # 一些错误 3 

       LOG_WARNING    warning conditions  # 不至于影响正常运转 4

       LOG_NOTICE     normal, but significant, condition # 5

       LOG_INFO       informational message # 6

       LOG_DEBUG      debug-level message # 7

       The function setlogmask(3) can be used to restrict logging to specified
       levels only.
       
# 注意： nome 表示不使用信息等级，即不记录这些信息
# *.*;mail.none 表示记录所有服务的所有信息，但不记录mail的信息
```
连接符：服务名和信息等级间的符号
```
.  只要出现比指定信息等级高的信息就记录，最常用
.=  只记录指定信息等级信息
.！ 仅不记录指定等级信息
```
log 目标地址可为：
* /var/log
* 用户名（显示给使用者）或 \* (显示给目前线上所有人)
* 打印设备 /dev/lp0 
* 远程主机
```
# 地址前面如果有 - 则表示先存在buffer中，数据量足够大再写入磁盘
mail.*                                                  -/var/log/maillog
```
### rsyslog 安全性
rsyslog 的 log 只要被编辑过就无法再记录新的数据，要让log可以重新被写入需要重启 rsyslog.service </br>
为提高安全性，可尝试设置让log 只能被增加数据，但不能被删除、移动</br>
注意：这么做会阻碍日志的分割备份
```
sudo chattr -a /var/log/admin.log
```
### rsyslog 服务器
用一台linux来管理多台linux的log，rsyslog 提供的端口为 UDP 或 TCP 的 port 514
* 服务端设定</br>
192.168.137.100
```
# 修改 /etc/rsyslog.conf

[sunnylinux@centOSlearning etc]$ sudo vim ./rsyslog.conf
# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514

# 重启 rsyslog
[sunnylinux@centOSlearning etc]$ sudo systemctl restart rsyslog.service

[sunnylinux@centOSlearning etc]$ sudo netstat -tnlup|grep 514
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      3504/rsyslogd
tcp6       0      0 :::514                  :::*                    LISTEN      3504/rsyslogd
```
* 客户端设定</br>
192.168.137.101
```
# 修改 /etc/rsyslog.conf
[sunnylinux@centOSlearning ~]$ sudo vim /etc/rsyslog.conf
# Send log to 192.168.137.100
*.*                                                     @@192.168.137.100

# 重启 rsyslog
[sunnylinux@centOSlearning etc]$ sudo systemctl restart rsyslog.service
```
注意：客户端的log信息会直接被记载在对应服务端的log文件中，可额外另起文件，要注意服务端防火墙配置，不然服务端可能收不到客户端的数据。

## logrotate 日志轮替
logrotate 功能：定时将当前log转为旧log，建立新log，可指定rotate的log个数，达到指定个数后将久log删除。</br>
```
[sunnylinux@centOSlearning etc]$ cd /var/log
[sunnylinux@centOSlearning log]$ ls
anaconda           cups                messages-20190204  spooler-20190127
audit              dmesg               messages-20190210  spooler-20190204
boot.log           dmesg.old           ntpstats           spooler-20190210
boot.log-20190129  firewalld           php-fpm            tallylog
boot.log-20190130  gdm                 pluto              tuned
boot.log-20190202  grubby              ppp                wpa_supplicant.log
boot.log-20190204  grubby_prune_debug  qemu-ga            wtmp
boot.log-20190205  httpd               rhsm               wtmp1
boot.log-20190208  lastlog             sa                 wtmp2
boot.log-20190211  maillog             samba              Xorg.0.log
btmp               maillog-20190120    secure             Xorg.0.log.old
btmp-20190201      maillog-20190127    secure-20190120    Xorg.1.log
chrony             maillog-20190204    secure-20190127    Xorg.1.log.old
cron               maillog-20190210    secure-20190204    Xorg.2.log
cron-20190120      mariadb             secure-20190210    Xorg.2.log.old
cron-20190127      messages            speech-dispatcher  Xorg.9.log
cron-20190204      messages-20190120   spooler            yum.log
cron-20190210      messages-20190127   spooler-20190120   yum.log-20190105

# 当前配置是一周轮替一次，保留4个backlog
```
rsyslog 是系统服务，而 logrotate 更像是个普通脚本，它依靠 crontab 来运行。</br>
```
Feb 11 13:07:02 centOSlearning run-parts(/etc/cron.daily)[2905]: starting logrotate
```
### logrotate 配置
logrotate 相关配置文件
```
[sunnylinux@centOSlearning etc]$ ls|grep logrotate
logrotate.conf  # 总体的配置文件
logrotate.d  # 更细致的配置文件
```
/etc/logrotate.d/\*.conf 中的细致配置会被读到 /etc/logrotate.conf 中执行
```
[sunnylinux@centOSlearning etc]$ cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly 
weekly

# keep 4 weeks worth of backlogs  
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file  设置被轮替档名加日期
dateext

# uncomment this if you want your log files compressed  是否压缩，log太大可考虑这个，除了httpd外大部分log不需要压缩
#compress

# RPM packages drop log rotation information into this directory 读取/etc/logrotate.d中的配置
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here 针对某个log设定参数
/var/log/wtmp {
    monthly
    create 0664 root utmp  # 指定新档案的权限、所有者、分组
        minsize 1M  # 超过1M才rotate，即是如果到达月rotate时间，但还未到1M依然不rotate
    rotate 1  # backlog 的数量
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```
更细致的log rotate 设置可以在/etc/logrotate.d/* 中完成
```
[sunnylinux@centOSlearning log]$ ls /etc/logrotate.d/bootlog
/etc/logrotate.d/bootlog
[sunnylinux@centOSlearning log]$ cat /etc/logrotate.d/bootlog
/var/log/boot.log  # 要处理档名，可同时设多个，一行写一档名
{
    missingok  
    daily
    copytruncate
    rotate 7
    notifempty
}
```
可在大括号内添加 sharedscripts .... endscript 以执行外部脚本命令
* prerotate 轮替前执行脚本
* postrotate 轮替后执行脚本</br>
可配合前面提到的 chattr +a 使用，rotate前取消a参数，rotate后再加上，提高安全性</br>

## 日志生成配置及轮替实例
将所有信息记录到 /var/log/admin.log 中，要求 +a 参数隐藏标签，每月轮替一次，大于10M时才轮替，保留5个backlog，压缩备份档。
```
# 配置 /etc/rsyslog.conf 生成日志
[sunnylinux@centOSlearning log]$ sudo cat /etc/rsyslog.conf
# Save info added by sharon 20190211
*.info                                                  /var/log/admin.log
[sunnylinux@centOSlearning log]$ sudo systemctl restart rsyslog.service
[sunnylinux@centOSlearning log]$ ls|grep admin
admin.log

# 配置/etc/logrotate.d/admin.conf 设置rotate
[sunnylinux@centOSlearning log]$ sudo vim /etc/logrotate.d/admin.conf
/var/log/admin.log{
    monthly
    rotate 5
    compress
    minsize 10M
    sharedscripts
    prerotate
        /usr/bin/chattr -a /var/log/admin.log  # 去掉 a 隐藏选项
    endscript
    sharedscripts
    postrotate
       /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true # reload一下rsyslog
       /usr/bin/chattr +a /var/log/admin.log
    endscript
}

# 给 /var/log/admin.log 上a参数，禁止减少数据、删除、移动 log
[sunnylinux@centOSlearning log]$ ll /var/log/admin.log
-rw-------. 1 root root 6147 2月  11 17:52 /var/log/admin.log
[sunnylinux@centOSlearning log]$ sudo chattr +a /var/log/admin.log
[sunnylinux@centOSlearning log]$ ll /var/log/admin.log
-rw-------. 1 root root 6278 2月  11 17:54 /var/log/admin.log
[sunnylinux@centOSlearning log]$ sudo lsattr /var/log/admin.log
-----a---------- /var/log/admin.log
[sunnylinux@centOSlearning log]$ sudo mv /var/log/admin.log ./test
mv: 无法将"/var/log/admin.log" 移动至"./test": 不允许的操作

# 测试一下 logrotate 功能
[sunnylinux@centOSlearning log]$ sudo logrotate -v /etc/logrotate.conf
reading config file /etc/logrotate.conf
including /etc/logrotate.d
reading config file admin.conf
# 省略部分输出
rotating pattern: /var/log/admin.log 10485760 bytes (5 rotations)
empty log files are rotated, old logs are removed
considering log /var/log/admin.log
  log does not need rotating (log size is below the 'size' threshold)
not running prerotate script, since no logs will be rotated
not running postrotate script, since no logs were rotated

# 上面显示不满足rotate条件，所以没有rotate也没有执行script，使用 -f 强制rotate一下
[sunnylinux@centOSlearning log]$ sudo logrotate -vf /etc/logrotate.conf
reading config file /etc/logrotate.conf
including /etc/logrotate.d
reading config file admin.conf

rotating pattern: /var/log/admin.log forced from command line (5 rotations)
empty log files are rotated, old logs are removed
considering log /var/log/admin.log
  log needs rotating
rotating log /var/log/admin.log, log->rotateCount is 5
dateext suffix '-20190211'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding old rotated logs failed
running prerotate script
fscreate context set to system_u:object_r:var_log_t:s0
renaming /var/log/admin.log to /var/log/admin.log-20190211
creating new /var/log/admin.log mode = 0600 uid = 0 gid = 0
running postrotate script
compressing log with: /bin/gzip
set default create context to system_u:object_r:var_log_t:s0

[sunnylinux@centOSlearning log]$ ls |grep admin
admin.log
admin.log-20190211.gz
```
## systemd-journal 收集开关机过程数据日志
rsyslog 开机之后才能开始记录日志信息，开关机过程可用systemd附带的systemd-journal来收集后交给rsyslog处理。</br>
数据会以二进制形式被写入/run/log中，重启会被清空并加入新的数据，即数据时间跨度为本次开机到关机。</br>
systemd-journal 会保存本次开机以来所有类型的日志讯息，关机时会从内存中调出给rsystem。</br>
```
[sunnylinux@centOSlearning /]$ ls /run/log/journal
c186467137644f039846081199c8b99d

[sunnylinux@centOSlearning /]$ journalctl --system
-- Logs begin at 日 2019-02-10 23:29:22 CST, end at 一 2019-02-11 18:24:24 CST. --
2月 10 23:29:22 centOSlearning.SharonLi systemd-journal[78]: Runtime journal is using 6.2M (
2月 10 23:29:22 centOSlearning.SharonLi kernel: Initializing cgroup subsys cpuset
2月 10 23:29:22 centOSlearning.SharonLi kernel: Initializing cgroup subsys cpu
2月 10 23:29:22 centOSlearning.SharonLi kernel: Initializing cgroup subsys cpuacct
2月 10 23:29:22 centOSlearning.SharonLi kernel: Linux version 3.10.0-862.2.3.el7.centos.plus
2月 10 23:29:22 centOSlearning.SharonLi kernel: ------------[ cut here ]------------
2月 10 23:29:22 centOSlearning.SharonLi kernel: WARNING: CPU: 0 PID: 0 at ./arch/x86/include
2月 10 23:29:22 centOSlearning.SharonLi kernel: Modules linked in:
2月 10 23:29:22 centOSlearning.SharonLi kernel: CPU: 0 PID: 0 Comm: swapper Not tainted 3.10
2月 10 23:29:22 centOSlearning.SharonLi kernel: Call Trace:
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3fd3d83>] dump_stack+0x16/0x18
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3a54d3a>] __warn+0xea/0x110
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3a45a6a>] ? native_flush_tlb_global+0x9a
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3a54e4a>] warn_slowpath_null+0x2a/0x30
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3a45a6a>] native_flush_tlb_global+0x9a/0
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d42f9359>] setup_arch+0xc3/0xf5a
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d3a57af7>] ? vprintk_emit+0x337/0x550
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d42f3885>] start_kernel+0xbe/0x3ea
2月 10 23:29:22 centOSlearning.SharonLi kernel:  [<d42f3380>] i386_start_kernel+0x12e/0x131
2月 10 23:29:22 centOSlearning.SharonLi kernel: ---[ end trace f68728a0d3053b52 ]---
2月 10 23:29:22 centOSlearning.SharonLi kernel: e820: BIOS-provided physical RAM map:
2月 10 23:29:22 centOSlearning.SharonLi kernel: BIOS-e820: [mem 0x0000000000000000-0x0000000
2月 10 23:29:22 centOSlearning.SharonLi kernel: BIOS-e820: [mem 0x000000000009f800-0x0000000
2月 10 23:29:22 centOSlearning.SharonLi kernel: BIOS-e820: [mem 0x00000000000ca000-0x0000000
2月 10 23:29:22 centOSlearning.SharonLi kernel: BIOS-e820: [mem 0x00000000000dc000-0x0000000
```
使用logger发送数据到log中
```
logger -p service_name.lv 'massage'

[sunnylinux@centOSlearning /]$ logger -p user.info 'Hello, testing the use of logger.'
# 输出最新两条log
[sunnylinux@centOSlearning /]$ journalctl -n 2
-- Logs begin at 日 2019-02-10 23:29:22 CST, end at 一 2019-02-11 18:38:36 CST. --
2月 11 18:30:01 centOSlearning.SharonLi CROND[4997]: (root) CMD (/usr/lib/sa/sa1 1 1)
2月 11 18:38:36 centOSlearning.SharonLi sunnylinux[5435]: Hello, testing the use of logger.
```
## 日志分析
可使用 logwatch 工具 https://sourceforge.net/projects/logwatch/files/
