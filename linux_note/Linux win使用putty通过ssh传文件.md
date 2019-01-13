# Linux win使用putty通过ssh传文件
## putty简介
PuTTY是一个Telnet、SSH、rlogin、纯TCP以及串行接口连接软件。随着Linux在服务器端应用的普及，Linux系统管理越来越依赖于远程。在各种远程登录工具中，Putty是出色的工具之一，Putty是免费的。同类型的软件还有 SecureCRT、Xshell、WINSCP，Xshell也是免费的。</br>

## win 上配置
* 下载安装putty
下载地址 https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
* ifconfig 在linux上找到linux主机的ip，输入到putty中
* 在putty的命令界面输入linux的登录用户和密码，如果linux已开ssh就会登录成功
注意：如果ssh

## win和linux互传文件
### win往linux传文件
打开win的cmd，按格式输入：</br>
pscp win上源文件路径 linux用户名@linux的ip:要存放的目的路径
```
pscp C:\Users\Tommy\Desktop\nex.txt root@192.168.137.100:/home/sunnylinux
```
cmd会要求输入root的密码</br>
注意：linux的用户如果权限不够会提示permission denied

### linux往win传文件
同样是在win上的cmd操作，格式：</br>
pscp linux用户名@linux的ip:要存的文件路径 win上要存放的目的路径
```
pscp  root@192.168.137.100:/home/sunnylinuxpythonscript.tar.gz C:\Users\Tommy\Desktop
```
由于putty不支持传目录，所以要传目录的话要记得先用tar打包压缩一下再传

