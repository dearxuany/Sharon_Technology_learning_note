# Linux daemons 启动与管理
## daemons 分类
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

## daemons 命名规则
service 名后面加 d


## daemons 启动脚本和启动方式
Daemon的启动需要执行文件、配置文件、执行环境。</br>
</br>
启动脚本:</br>
进行环境的检测、加载提供的函数功能、配置文件的分析、PID文件的放置，以及重要交换文件的锁住 lock 操作。</br>
过去，系统服务的进程大多依靠一个init的启动机制来启动，CentOS7以后使用的是 systemd。</br>
### init
init方式启动，主要涉及到/etc 和 /var 下的一些文件，[linux 目录结构](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.md#etc)。</br>
```
/etc/init.d/*    启动脚本放置处，管理一个或多个daemon的启动（以前使用）
/etc/sysconfig/*  各服务的初始化环境配置文件
/etc/xinetd.conf  super daemon 默认配置文件
/etc/xinetd.d/*   super daemon 更细致的配置文件

/var/run/  启动的daemon进程PID记录处
/var/lib/* 各服务产生的数据库
```
### systemd
systemd 特点：</br>
* 速度比 init 快，systemd 本身常驻于内存
* systemd 服务仅需使用 systemctl 命令来处理，即是不用 init, chkconfig, service 来支撑，但减少了自定义的开放性
* systemd 将服务分为一个服务单位 unit，再将unit按功能分为type，type 有 service, socket, target, path, snapshot, timer
* init 有 runlevel，而systemd 对应的是 target ，但没有全部对应，runlevel依然无可取代
* systemd 向下兼容 init


## stand_alone daemons
### stand_alone daemons 启动
#### init 启动
CentOS7 后不用了，ubuntu 貌似还可以用
```
# 一般格式为
/etc/init.d/crond start
```
参考 [Linux(ubuntu)-Nginx-MySQL-PHP 架构 apt-get 安装配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/server_note/Nginx_note/Linux%20Nginx%20MySQL%20PHP%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.MD#%E5%B8%B8%E7%94%A8-nginx-%E5%91%BD%E4%BB%A4)
#### service 方式
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
systemctl 方式

## super daemons 
### super daemons 分类
根据处理方式分：
* 单线程
* 多线程


