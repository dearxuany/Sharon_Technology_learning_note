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
#### 使用 groups 命令 
每次新建用户如果不指定用户组的话，默认会自动创建一个与用户名相同的用户组
```
#  查看用户所属用户组方法1
[sunnylinux@centOSlearning ~]$ groups sunnylinux
sunnylinux : sunnylinux wheel
[sunnylinux@centOSlearning ~]$ groups sharonli
sharonli : sharonli
```
#### 查看 /etc/group 文件
/etc/group 的内容包括用户组（Group）、用户组口令、GID 及该用户组所包含的用户（User），每个用户组一条记录。
```
group_name:password:GID:user_list
```
注意：如果用户主用户组，即用户的 GID 等于用户组的 GID，那么最后一个字段 user_list 就是空的，因为这个用户组是默认创建的
```
#  查看用户所属用户组方法2
[sunnylinux@centOSlearning ~]$ cat /etc/group  # 省略部分输出
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:sunnylinux

[sunnylinux@centOSlearning ~]$ cat /etc/group|grep sunnylinux
wheel:x:10:sunnylinux
sunnylinux:x:1000:sunnylinux
sunnylinux2:x:1001:sunnylinux2

[sunnylinux@centOSlearning ~]$ cat /etc/group|grep sharonli
sharonli:x:1004:

```
### 加入 sudo 用户组
默认情况下在 sudo 用户组里的用户可以使用 sudo 命令获得 root 权限。</br>
#### /etc/sudoers 文件
CentOS 默认普通用户是无法使用sudo命令的，需要将登陆的用户加入/etc/sudoers 文件中。</br>
ubuntu 可在 /etc/sudoers.d 目录下，查看用户的对应文件 /etc/sudoers.d/用户名，如果存在则该用户有sudo权限。</br>
注意：一般情况下不会通过修改 /etc/sudoers 文件 来让一般用户拥有 sudo 权限，而是使用 usermod 命令为用户添加用户组
```
[sunnylinux@centOSlearning ~]$ sudo vim /etc/sudoers  # 也可用 sudo visudo

#
# Adding HOME to env_keep may enable a user to run unrestricted
# commands via sudo.
#
# Defaults   env_keep += "HOME"

Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin

## Next comes the main part: which users can run what software on
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

```
让一般用户获得sudo权限的修改方法
```
# 在这行下面添加一行
用户名  可执行命令的主机名或地址=（可使用的身份）  可执行的命令的绝对路径
root    ALL=(ALL)       ALL
用户名   ALL=(ALL)      ALL

# 对于组，注意在组名前面添加一个%
%组名  可执行命令的主机名或地址=（可使用的身份）  可执行的命令的绝对路径
%wheel  ALL=(ALL)       ALL
```
可执行的命令写得越详细越安全，普通用户执行该命令也只能严格输入命令的所有字符，即要使用绝对路径
```
# 让sharonli 在任何主机可切换到任何用户执行重启命令，shutdown 有很多选项，把命令写得越详细，执行的命令限制越多，例如这样用户就不能关机只能重启
sharonli ALL=(ALL)    /usr/sbin/shutdown -r now
# 一般用户执行方法
sudo /usr/sbin/shutdown -r now

# 让 sharonli 可添加用户，不指定可切换的用户则默认使用root
sharonli ALL=/usr/sbin/useradd
```
```
[sunnylinux@centOSlearning test]$ su sharonli
密码：
[sharonli@centOSlearning test]$ sudo -l
[sudo] sharonli 的密码：
匹配 %2$s 上 %1$s 的默认条目：
    !visiblepw, always_set_home, match_group_by_gid, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME
    LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

用户 sharonli 可以在 centOSlearning 上运行以下命令：
    (ALL) /usr/sbin/shutdown -r now
    (root) /usr/sbin/useradd
[sharonli@centOSlearning test]$ useradd abc
bash: /usr/sbin/useradd: 权限不够
[sharonli@centOSlearning test]$ sudo useradd abc
[sharonli@centOSlearning test]$ ls /home
abc  mysql  sharonli  sunnylinux  sunnylinux2  tommy  www
[sharonli@centOSlearning test]$ sudo userdel abc
对不起，用户 sharonli 无权以 root 的身份在 centOSlearning.SharonLi 上执行 /sbin/userdel abc。

```
sunnylinux 在 wheel 里，可设置不用输密码（此处没有设），用户可查看自己可以执行的 sudo 命令
```
[sunnylinux@centOSlearning test]$ sudo -l
[sudo] sunnylinux 的密码：
匹配 %2$s 上 %1$s 的默认条目：
    !visiblepw, always_set_home, match_group_by_gid, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME
    LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

用户 sunnylinux 可以在 centOSlearning 上运行以下命令：
    (ALL) ALL
    
[sunnylinux@centOSlearning test]$ userdel abc
bash: /usr/sbin/userdel: 权限不够
[sunnylinux@centOSlearning test]$ sudo userdel abc
[sudo] sunnylinux 的密码：
```
对于其他没有 sudo 权限的一般用户
```
[sunnylinux@centOSlearning ~]$ su tommy
密码：
[tommy@centOSlearning sunnylinux]$ sudo ls

我们信任您已经从系统管理员那里了解了日常注意事项。
总结起来无外乎这三点：

    #1) 尊重别人的隐私。
    #2) 输入前要先考虑(后果和风险)。
    #3) 权力越大，责任越大。

[sudo] tommy 的密码：
tommy 不在 sudoers 文件中。此事将被报告。
```

#### usermod 命令
使用 usermod 命令可以为用户添加用户组
```
# 添加用户到 wheel 用户组，让该用户获得 sudo 权限
[sunnylinux@centOSlearning ~]$ groups sharonli
sharonli : sharonli
[sunnylinux@centOSlearning ~]$ sudo usermod -G wheel sharonli
[sudo] sunnylinux 的密码：
[sunnylinux@centOSlearning ~]$ groups sharonli
sharonli : sharonli wheel

[sunnylinux@centOSlearning ~]$ su sharonli
密码：
[sharonli@centOSlearning sunnylinux]$ sudo ls /home
[sudo] sharonli 的密码：
mysql  sharonli  sunnylinux  sunnylinux2  www
```
### 删除用户
userdel 删除用户，userdel只能删除用户，并不会删除相关的目录文件。userdel -r 可以删除用户及相关目录。</br>
```
[sunnylinux@centOSlearning ~]$ sudo userdel -r sharonli
[sudo] sunnylinux 的密码：
userdel: user sharonli is currently used by process 4480
# 需要强制退出
[sunnylinux@centOSlearning ~]$ ps -aux|grep 4480
sharonli  4480  0.0  0.2   7932  2496 pts/1    S    2月05   0:00 bash
sunnyli+  4885  0.0  0.0   6704   880 pts/1    R+   00:02   0:00 grep --color=auto 4480
[sunnylinux@centOSlearning ~]$ sudo kill -9 4480
[sunnylinux@centOSlearning ~]$ 已杀死
[sunnylinux@centOSlearning ~]$ sudo userdel sharonli -r
[sudo] sunnylinux 的密码：
[sunnylinux@centOSlearning ~]$ groups sharonli
groups: sharonli: no such user

```
## 文件权限
### 查看文件权限
```
[sunnylinux@centOSlearning test]$ ls -al
总用量 8
drwxrwxr-x.  8 sunnylinux sunnylinux  185 2月   4 12:28 .
drwx------. 34 sunnylinux sunnylinux 4096 2月   5 15:15 ..
drwxrwxr-x.  2 sunnylinux sunnylinux   21 3月  16 2018 looptest
drwxrwxr-x.  2 sunnylinux sunnylinux   88 3月   6 2018 mvtest
drwxrwxr-x.  5 sunnylinux sunnylinux 4096 5月  28 2018 scripttest
drwxrwxr-x.  5 sunnylinux sunnylinux   93 2月   4 12:56 tartest
drwxrwxr-x.  3 sunnylinux sunnylinux   18 5月  28 2018 testing
-rw-rw-r--.  1 root       root          0 3月   6 2018 umasktest2.txt
-rw-r--r--.  1 root       root          0 3月   6 2018 umasktest3.txt
-rw-rw-r--.  1 root       root          0 3月   6 2018 umasktest4.txt
-rw-r--r--.  1 root       root          0 3月   6 2018 umasktest.txt
drwxrwxr-x.  2 sunnylinux sunnylinux   56 9月  18 09:30 vitest
```
drwxrwxr-x 文件属性、拥有者权限、用户组权限、其他用户权限 755 rwxr-xr-x </br>
rwx 对应 421 即 2^2+2^1+2^0=7 </br>
目录可进入、内部文件可查看必须有r和x权限，可在目录新建文件必须有w权限</br>

### 改变文件拥有者和用户组 chown
chown   修改每个由第一个非选项参数声明的给定file(文件)的用户和/或组的所有权.如下:</br>
如果只给出了用户名(或者数字用户标识),那么该用户即成为每个指定文件的所有者,而该文件的组别并不改变.如果用户名后面紧跟着冒号和组名(或者是数字组标识),并且它们之间没有空格,那么文件的组所有权也随之改变.
```
[sunnylinux@centOSlearning test]$ sudo touch owner.txt
[sudo] sunnylinux 的密码：
[sunnylinux@centOSlearning test]$ ls -al|grep owner
-rw-r--r--.  1 root       root          0 2月   6 00:50 owner.txt

[sunnylinux@centOSlearning test]$ sudo chown sunnylinux ./owner.txt
[sunnylinux@centOSlearning test]$ ls -l ./owner.txt
-rw-r--r--. 1 sunnylinux root 0 2月   6 00:50 ./owner.txt

[sunnylinux@centOSlearning test]$ chown sunnylinux:sunnylinux ./owner.txt
[sunnylinux@centOSlearning test]$ ls -l ./owner.txt
-rw-r--r--. 1 sunnylinux sunnylinux 0 2月   6 00:50 ./owner.txt
```
### 改变文件权限 chmod
```
[sunnylinux@centOSlearning test]$ chmod 755 ./owner.txt
[sunnylinux@centOSlearning test]$ ls -l ./owner.txt
-rwxr-xr-x. 1 sunnylinux sunnylinux 0 2月   6 00:50 ./owner.txt
```
