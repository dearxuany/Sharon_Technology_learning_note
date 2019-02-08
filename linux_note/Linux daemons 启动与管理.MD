# Linux daemons 启动与管理
## daemons 分类
按照 daemons 的启动和管理方式分类：</br>
* stand_alone</br>
常驻于内存当中，可自启，不需要额外的管理，响应速度快。</br>
例子：httpd、vsftpd</br>
* super daemons</br>
由一个总管进程 xinetd 同一管理，有请求来时才启动，不会一直占用内存空间，启动较慢，可以具有安全控管的机制，提供IP来源、时间、连线管理等功能。</br>
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
启动脚本:进行环境的检测、加载提供的函数功能、配置文件的分析、PID文件的放置，以及重要交换文件的锁住 lock 操作。</br>
主要涉及到/etc 和 /var 下的一些文件，[linux 目录结构](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.md#etc)。</br>
```
/etc/init.d/*    启动脚本放置处，管理一个或多个daemon的启动
/etc/sysconfig/*  各服务的初始化环境配置文件
/etc/xinetd.conf  super daemon 默认配置文件
/etc/xinetd.d/*   super daemon 更细致的配置文件

/var/run/  启动的daemon进程PID记录处
/var/lib/* 各服务产生的数据库
```

## super daemons 
### super daemons 分类
根据处理方式分：
* 单线程
* 多线程


