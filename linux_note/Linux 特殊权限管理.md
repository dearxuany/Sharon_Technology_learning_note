# Linux 特殊权限管理

## ACL 权限
一个文件或目录只能有一个 owner 权限和 一个 group 权限以及其他用户权限，这就导致有时权限不够用，于是出现了特殊权限 ACL。
例如，一个目录权限是 drwxrwxr-- ，但有其中一个用户需要权限r-x且该用户不是拥有者，也不在group内，这时就使用ACL设置。

### 开启ACL权限
CentOS7 ACL 权限默认开启，作用于挂载分区</br>
相关命令：</br>
* dmesg</br>
用来显示开机信息，kernel会将开机信息存储在ring buffer中。若是开机时来不及查看信息，可利用dmesg来查看。</br>
开机信息亦保存在/var/log目录中，名称为dmesg的文件里。</br>
* xfs_growfs</br>
因为文件系统是 xfs 所以和 ext 的命令是不一样的，用不到 dumpe2fs，要用 xfs_growfs。</br>
```
# 查看系统分区
$ df
文件系统                   1K-块     已用     可用 已用% 挂载点
/dev/mapper/centos-root 25155584 12258844 12896740   49% /
devtmpfs                  498448        0   498448    0% /dev
tmpfs                     513800        0   513800    0% /dev/shm
tmpfs                     513800     8052   505748    2% /run
tmpfs                     513800        0   513800    0% /sys/fs/cgroup
/dev/sda2                1038336   208212   830124   21% /boot
/dev/mapper/centos-home  5232640  1293612  3939028   25% /home
tmpfs                     102760       12   102748    1% /run/user/42
tmpfs                     102760        0   102760    0% /run/user/0
tmpfs                     102760        0   102760    0% /run/user/1000

# 
[sunnylinux@centOSlearning ~]$ dmesg|grep -i acl
[    2.541023] systemd[1]: systemd 219 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 -SECCOMP +BLKID +ELFUTILS +KMOD +IDN)
[    4.302231] SGI XFS with ACLs, security attributes, no debug enabled
```
设置 acl 长期启动，修改/etc/fstab </br> 
defaults 表示挂载时使用默认权限，默认权限中已包含ACL权限。</br>
如果要添加也可改为 defaults,acl，修改配置之后要重新挂载一下该分区使配置生效。</br>
```
# /etc/fstab
# Created by anaconda on Sun Feb 25 21:37:18 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=b4cb2180-3d2f-4fb7-84c6-0e994fc5fd64 /boot                   xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```
### 设置 ACL 权限
```
[sunnylinux@centOSlearning ~]$ setfacl --help
setfacl 2.2.51 -- set file access control lists
Usage: setfacl [-bkndRLP] { -m|-M|-x|-X ... } file ...
  -m, --modify=acl        modify the current ACL(s) of file(s)  设置ACL权限
  -x, --remove=acl        remove entries from the ACL(s) of file(s)  删除指定文件的ACL权限
  -b, --remove-all        remove all extended ACL entries 删除所有ACL权限  
  -k, --remove-default    remove the default ACL   删除默认ACL权限
      --set=acl           set the ACL of file(s), replacing the current ACL
      --set-file=file     read ACL entries to set from file
      --mask              do recalculate the effective rights mask
  -n, --no-mask           don't recalculate the effective rights mask
  -d, --default           operations apply to the default ACL 使用默认ACL权限
  -R, --recursive         recurse into subdirectories 递归设定ACL权限
  -L, --logical           logical walk, follow symbolic links
  -P, --physical          physical walk, do not follow symbolic links
      --restore=file      restore ACLs (inverse of `getfacl -R')
      --test              test mode (ACLs are not modified)
  -v, --version           print version and exit
  -h, --help              this help text
```
#### 添加ACL与查看
ACL 实例
```
# 有一文件属于 sharonli 和 student 用户组
drwxrwx---.  2 sharonli   student       6 2月  12 16:24 acltest

# 由于当前账户不在 student 组内所以无法进入该目录
[sunnylinux@centOSlearning test]$ cd ./acltest/
-bash: cd: ./acltest/: 权限不够

# 在不用sudo前提下，用ACL让本账号拥有进如acltest的权限
[sunnylinux@centOSlearning test]$ sudo setfacl -m u:sunnylinux:rx ./acltest/  # u 表用户：用户名：权限，用户组设定用 g

# 可看到当前用户在该目录的权限后面多了个+号
drwxrwx---+  2 sharonli   student       6 2月  12 16:24 acltest

# 查看ACL权限
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
group:student2:r-x
mask::rwx
other::---

[sunnylinux@centOSlearning test]$ cd ./acltest/
[sunnylinux@centOSlearning acltest]$ pwd
/home/sunnylinux/test/acltest
```

#### ACL 最大有效权限
用户所获得的ACL真实权限是 setfacl 设置的权限 和 mask 权限 做“与运算”所获得的权限，一般不用修改 
```
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
group:student2:r-x
mask::rwx
other::---
```
可用 setfctl -m m:权限 filename 来修改 mask 权限
注意：mask 会影响到文件的所属组权限，同时如果重新设u或g的ACL，mask会受影响
```
[sunnylinux@centOSlearning test]$ sudo setfacl -m m:rw ./acltest/
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x             #effective:r--
group::rwx                      #effective:rw-
group:student2:r-x              #effective:r--
mask::rw-
other::---

[sunnylinux@centOSlearning test]$ sudo setfacl -m u:sunnylinux:rx ./acltest/
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
group:student2:r-x
mask::rwx  # mask被恢复了
other::---

```
#### 删除 ACL 权限
删除指定用户或组的 acl 权限
```
[sunnylinux@centOSlearning test]$ sudo setfacl -x g:student2 ./acltest/
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
mask::rwx
other::---
```
删除某文件或目录指定的所有ACL权限
```
[sunnylinux@centOSlearning test]$ sudo setfacl -b ./acltest/
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
group::rwx
other::---
```
#### ACL 递归权限
在设置某目录的ACL权限同时给目录内的文件也设置ACL权限
```
[sunnylinux@centOSlearning test]$ getfacl ./acltest/
# file: acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
mask::rwx
other::---

[sunnylinux@centOSlearning acltest]$ ls -al  
总用量 0
drwxrwx---+ 2 sharonli   student     38 2月  12 17:38 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r--r--. 1 root       root         0 2月  12 17:38 acl0.txt  # 子文件是没有ACL权限的
-rw-r--r--. 1 root       root         0 2月  12 17:38 acl1.txt

[sunnylinux@centOSlearning test]$ sudo setfacl -m u:sunnylinux:rx -R ./acltest/  # 要 -m 和 -R 一起用
[sunnylinux@centOSlearning test]$ cd ./acltest/
[sunnylinux@centOSlearning acltest]$ ls -al
总用量 0
drwxrwx---+ 2 sharonli   student     38 2月  12 17:38 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl0.txt
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl1.txt
```
文件和目录的mask权限是不一样，这是系统默认设定的
```
[sunnylinux@centOSlearning acltest]$ getfacl acl0.txt
# file: acl0.txt
# owner: root
# group: root
user::rw-
user:sunnylinux:r-x
group::r--
mask::r-x
other::r--
```
关于权限溢出：</br>
对于目录来说，执行权限代表可进入，但对于文件来说，执行权限代表文件可运行，这让文件获得了过大的权限。</br>
使用ACL的-R参数，让目录和文件同时获得x权限，让文件的权限过大，这个问题称为“权限溢出”，难以避免。</br>

#### 默认权限
一旦给父目录设置了默认ACL权限，则其之后在目录内新建的文件和目录均含有指定的ACL权限
```
[sunnylinux@centOSlearning acltest]$ ls -al
总用量 0
drwxrwx---+ 2 sharonli   student     38 2月  12 17:38 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl0.txt
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl1.txt

# 新建一个文件
[sunnylinux@centOSlearning acltest]$ ls -al
总用量 0
drwxrwx---+ 2 sharonli   student     54 2月  12 17:54 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl0.txt
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl1.txt
-rw-r--r--. 1 root       root         0 2月  12 17:54 acl2.txt  # 新建的ACL是没有ACL权限的
```
给父目录设置默认权限
```
[sunnylinux@centOSlearning acltest]$ sudo setfacl -m d:u:sunnylinux:rx ../acltest/  # 使用d参数，要和 -m 一起用
[sunnylinux@centOSlearning acltest]$ getfacl ../acltest/
# file: ../acltest/
# owner: sharonli
# group: student
user::rwx
user:sunnylinux:r-x
group::rwx
mask::rwx
other::---
default:user::rwx
default:user:sunnylinux:r-x
default:group::rwx
default:mask::rwx
default:other::---

[sunnylinux@centOSlearning acltest]$ ls -al
总用量 0
drwxrwx---+ 2 sharonli   student     54 2月  12 17:54 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl0.txt
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl1.txt
-rw-r--r--. 1 root       root         0 2月  12 17:54 acl2.txt # 这个文件还是没有ACL权限的

# 新建一文件
[sunnylinux@centOSlearning acltest]$ sudo touch ./acl3.txt
[sunnylinux@centOSlearning acltest]$ ls -al
总用量 0
drwxrwx---+ 2 sharonli   student     70 2月  12 18:00 .
drwxrwxr-x. 9 sunnylinux sunnylinux 217 2月  12 16:24 ..
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl0.txt
-rw-r-xr--+ 1 root       root         0 2月  12 17:38 acl1.txt
-rw-r--r--. 1 root       root         0 2月  12 17:54 acl2.txt
-rw-rw----+ 1 root       root         0 2月  12 18:00 acl3.txt  # 没有变成可执行文件，但获得了ACL权限
[sunnylinux@centOSlearning acltest]$ getfacl acl3.txt
# file: acl3.txt
# owner: root
# group: root
user::rw-
user:sunnylinux:r-x             #effective:r--
group::rwx                      #effective:rw-
mask::rw-  # 注意这里！
other::---

```


