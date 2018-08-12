# VMware 虚拟机扩容
给VMware上的虚拟机扩容有两部分设置，一部分在物理机的VMware软件上，另外一部分是在虚拟机内部的设置。</br>
对于系统为linux的虚拟机，虚拟机部分的设置主要涉及LVM的设置流程。

## VMware 物理机上的操作
注意：</br>
linux只能扩展磁盘容量而不能减小(xfs文件系统)，所填写的容量为总容量，即包含已分区的磁盘；</br>
扩展容量时不能有快照，可以先把快照删除掉；</br>
扩展容量前要先关闭虚拟机。</br>
</br>
在VMware软件内操作：</br>
VM -> Settings... -> Hardware -> Hard Disk -> Utilities -> Expand</br>
输入想扩展到的磁盘大小后，按Expand。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A9%E5%AE%B91.png)
</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%89%A9%E5%AE%B92.png)</br>
至此，物理机上的设置就完成了。

## VMware 虚拟机上操作
开启虚拟机，登录root账号

### 查看磁盘使用状况
常用到以下几条命令来分析磁盘状况
#### df 报告文件系统磁盘空间的使用情况
```
# df -hT
文件系统                类型      容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root xfs        24G  9.7G   15G   41% /
devtmpfs                devtmpfs  487M     0  487M    0% /dev
tmpfs                   tmpfs     502M     0  502M    0% /dev/shm
tmpfs                   tmpfs     502M  8.1M  494M    2% /run
tmpfs                   tmpfs     502M     0  502M    0% /sys/fs/cgroup
/dev/sda2               xfs      1014M  204M  811M   21% /boot
/dev/mapper/centos-home xfs       5.0G  1.1G  4.0G   22% /home
tmpfs                   tmpfs     101M  8.0K  101M    1% /run/user/42
tmpfs                   tmpfs     101M   24K  101M    1% /run/user/1000
```
主要注意的是各个目录所用的文件系统类型以及它们所占空间及剩余空间。</br>

#### lsblk 列出系统上所有磁盘的列表
```
# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk 
sda               8:0    0   60G  0 disk 
├─sda1            8:1    0    2M  0 part 
├─sda2            8:2    0    1G  0 part /boot
├─sda3            8:3    0   30G  0 part 
│ ├─centos-root 253:0    0   10G  0 lvm  /
│ ├─centos-swap 253:1    0    1G  0 lvm  [SWAP]
│ └─centos-home 253:2    0    5G  0 lvm  /home
├─sda4            8:4    0    1K  0 part 
├─sda5            8:5    0    2G  0 part /data/xfs
├─sda6            8:6    0  954M  0 part /data/ext4
└─sda7            8:7    0  1.9G  0 part 
sr0              11:0    1 1024M  0 rom  
```
主要注意各个目录的挂载点以及各个磁盘类型。</br>
根目录是在/dev/sda3，sda5、sda6、sda7是扩展分区sda4上的逻辑分区。

#### 查看磁盘的分割方式
现时linux上的磁盘分割方式有两种：GPT和MBR。</br>
MBR的分割方式比GPT要旧一点，MBR分割方式的磁盘大小小于2T，所以现在一般用GPT。</br>
扩容之前一定要查一查分割方式，因为会用到的命令不一样，MBR用的是fdisk，GPT用的是gdisk，如果命令用错了会造成磁盘损坏！！！</br>
</br>
parted 列出磁盘分割表类型和分割资信
```
# parted /dev/sda print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sda: 64.4GB  # 物理磁碟容量（此处指的是VMware给虚拟机分配的磁盘大小）
Sector size (logical/physical): 512B/512B
Partition Table: msdos  # 分割表的格式，msdos貌似是一种虚拟MBR，如果是GPT显示gpt
Disk Flags: 

Number  Start   End     Size    Type      File system  标志
 1      1049kB  3146kB  2097kB  primary
 2      3146kB  1077MB  1074MB  primary   xfs          启动
 3      1077MB  33.3GB  32.2GB  primary                lvm
 4      33.3GB  42.9GB  9656MB  extended
 5      33.3GB  35.4GB  2147MB  logical   xfs
 6      35.4GB  36.4GB  1000MB  logical   ext4
 7      36.7GB  38.7GB  2000MB  logical
```
注意磁盘的分割方式，所以之后的用于扩容的命令也就选用fdisk而不是gdisk了。

### 虚拟机磁盘扩容
要根据磁盘的状态来决定之后的扩容操作，主要涉及到linux上的LVM。</br>
</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/Linux%20LVM%E6%B5%81%E7%A8%8B.png)
