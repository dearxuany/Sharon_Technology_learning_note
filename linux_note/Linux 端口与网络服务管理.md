# Linux 端口与网络服务管理
## 查看网络服务接口
查看网络服务开启的端口
注意：需要root权限才能看到 -p 的内容
```
[sunnylinux@centOSlearning system]$ sudo netstat -tunlp
[sudo] sunnylinux 的密码：
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1671/mysqld
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      4401/nginx: master
tcp        0      0 0.0.0.0:1234            0.0.0.0:*               LISTEN      4401/nginx: master
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1189/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1650/master
tcp6       0      0 :::22                   :::*                    LISTEN      1189/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1650/master
udp        0      0 0.0.0.0:43987           0.0.0.0:*                           682/avahi-daemon: r
udp        0      0 192.168.137.100:123     0.0.0.0:*                           708/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           708/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           708/ntpd
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           682/avahi-daemon: r
udp6       0      0 fe80::fade:7bb1:bd5:123 :::*                                708/ntpd
udp6       0      0 ::1:123                 :::*                                708/ntpd
udp6       0      0 :::123                  :::*                                708/ntpd
-       
```
不用 -n　参数的时候显示的是服务名而不是端口号
```
[sunnylinux@centOSlearning system]$ sudo netstat -tul
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:mysql           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:search-agent    0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN
udp        0      0 0.0.0.0:43987           0.0.0.0:*
udp        0      0 centOSlearning.Shar:ntp 0.0.0.0:*
udp        0      0 localhost:ntp           0.0.0.0:*
udp        0      0 0.0.0.0:ntp             0.0.0.0:*
udp        0      0 0.0.0.0:mdns            0.0.0.0:*
udp6       0      0 centOSlearning.Shar:ntp [::]:*
udp6       0      0 localhost:ntp           [::]:*
udp6       0      0 [::]:ntp                [::]:*

```
## 查看对应端口的网络服务
```
[sunnylinux@centOSlearning system]$ systemctl status avahi-daemon
● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
   Loaded: loaded (/usr/lib/systemd/system/avahi-daemon.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2019-02-08 09:42:09 CST; 8h ago
 Main PID: 682 (avahi-daemon)
   Status: "avahi-daemon 0.6.31 starting up."
   CGroup: /system.slice/avahi-daemon.service
           ├─682 avahi-daemon: running [centOSlearning.local]
           └─701 avahi-daemon: chroot helper

2月 08 09:44:34 centOSlearning.SharonLi avahi-daemon[682]: Registering new address record for 192.168.137.100 on ens33.IPv4.
2月 08 09:44:36 centOSlearning.SharonLi avahi-daemon[682]: Registering new address record for fe80::fade:7bb1:bd57:33e...3.*.
2月 08 14:19:19 centOSlearning.SharonLi avahi-daemon[682]: Withdrawing address record for 192.168.137.100 on ens33.
2月 08 14:19:19 centOSlearning.SharonLi avahi-daemon[682]: Leaving mDNS multicast group on interface ens33.IPv4 with a...100.
2月 08 14:19:19 centOSlearning.SharonLi avahi-daemon[682]: Interface ens33.IPv4 no longer relevant for mDNS.
2月 08 14:19:19 centOSlearning.SharonLi avahi-daemon[682]: Withdrawing address record for fe80::fade:7bb1:bd57:33e7 on ens33.
2月 08 14:19:33 centOSlearning.SharonLi avahi-daemon[682]: Joining mDNS multicast group on interface ens33.IPv4 with a...100.
2月 08 14:19:33 centOSlearning.SharonLi avahi-daemon[682]: New relevant interface ens33.IPv4 for mDNS.
2月 08 14:19:33 centOSlearning.SharonLi avahi-daemon[682]: Registering new address record for 192.168.137.100 on ens33.IPv4.
2月 08 14:19:34 centOSlearning.SharonLi avahi-daemon[682]: Registering new address record for fe80::fade:7bb1:bd57:33e...3.*.
Hint: Some lines were ellipsized, use -l to show in full.
```
查看其相关联的服务
```
[sunnylinux@centOSlearning system]$ systemctl| grep avahi-daemon
  avahi-daemon.service                                                                loaded active running   Avahi mDNS/DNS-SD Stack
  avahi-daemon.socket                                                                 loaded active running   Avahi mDNS/DNS-SD Stack Activation Socket

```
可按需将其关闭（stop和disabled）
