# Linux 用户及文件权限管理
## 用户
### 查看当前登录用户信息
```
$ whoami
sunnylinux

# 当前登录用户数及用户名
$ who -q
sunnylinux sunnylinux
# 用户数=2

# 打印所有信息
$ who -a
           系统引导 2019-02-05 15:07
           运行级别 5 2019-02-05 15:10
sunnylinux + pts/0        2019-02-05 15:13   .          2078 (tommy-pc.mshome.net)
sunnylinux ? :0           2019-02-05 15:15   ?          2162 (:0)

```
## 创建用户
一般登录系统时都是以普通账户的身份登录的，要创建用户需要 root 权限。
可使用sudo命令让一般用户暂时获得root权限，使用这个命令有两个大前提，一是你要知道当前登录用户的密码，二是当前用户必须在 sudo 用户组。
```
# 创建一个新用户
[sunnylinux@centOSlearning ~]$ sudo adduser sharonli

# 给新用户设置密码
[sunnylinux@centOSlearning ~]$ sudo passwd sharonli
更改用户 sharonli 的密码 。
新的 密码：
无效的密码： 密码包含用户名在某些地方
重新输入新的 密码：
抱歉，密码不匹配。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```
useradd与adduser都是创建新的用户</br>
在CentOs下useradd与adduser是没有区别的都是在创建用户，在home下自动创建目录，没有设置密码，需要使用passwd命令修改密码。</br>
而在Ubuntu下useradd与adduser有所不同</br>
1、useradd在使用该命令创建用户是不会在/home下自动创建与用户名同名的用户目录，而且不会自动选择shell版本，也没有设置密码，那么这个用户是不能登录的，需要使用passwd命令修改密码。</br>
2、adduser在使用该命令创建用户是会在/home下自动创建与用户名同名的用户目录，系统shell版本，会在创建时会提示输入密码，更加友好。</br>
userdel 删除用户，userdel只能删除用户，并不会删除相关的目录文件。userdel -r 可以删除用户及相关目录。</br>
```
# adduser默认为新用户创建 home 目录
[sunnylinux@centOSlearning ~]$ ls /home
mysql  sharonli  sunnylinux  sunnylinux2  www

# 切换用户
[sunnylinux@centOSlearning ~]$ su sharonli
密码：
[sharonli@centOSlearning sunnylinux]$ ls
ls: 无法打开目录.: 权限不够
```
## 用户组
### 查看用户所属用户组
每次新建用户如果不指定用户组的话，默认会自动创建一个与用户名相同的用户组
```
[sunnylinux@centOSlearning ~]$ groups sunnylinux
sunnylinux : sunnylinux wheel
[sunnylinux@centOSlearning ~]$ groups sharonli
sharonli : sharonli
```
默认情况下在 sudo 用户组里的可以使用 sudo 命令获得 root 权限。</br>
CentOS 
ubuntu 可在 /etc/sudoers.d 目录下，查看用户的对应文件 /etc/sudoers.d/用户名，如果存在则该用户有sudo权限。</br>

```
