# Linux 端口与网络服务管理
## 查看网络服务接口
```
[sunnylinux@centOSlearning system]$ netstat -tlunp
(No info could be read for "-p": geteuid()=1000 but you should be root.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      -                
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                
tcp6       0      0 :::22                   :::*                    LISTEN      -                
tcp6       0      0 ::1:25                  :::*                    LISTEN      -                
udp        0      0 0.0.0.0:43987           0.0.0.0:*                           -                
udp        0      0 192.168.137.100:123     0.0.0.0:*                           -                
udp        0      0 127.0.0.1:123           0.0.0.0:*                           -                
udp        0      0 0.0.0.0:123             0.0.0.0:*                           -                
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           -                
udp6       0      0 fe80::fade:7bb1:bd5:123 :::*                                -                
udp6       0      0 ::1:123                 :::*                                -                
udp6       0      0 :::123                  :::*                                -       
```
