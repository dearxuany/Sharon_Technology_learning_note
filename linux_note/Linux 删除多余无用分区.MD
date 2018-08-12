# Linux 删除多余无用分区
起因：</br>
之前linux学习的时候建了很多不同文件系统类型的无用小分区，现在想删了它们，让系统更易于管理。</br>

## 查看磁盘使用状况
### df 报告文件系统磁盘空间的使用情况
```
#  df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   10G  9.7G  363M   97% /
devtmpfs                 487M     0  487M    0% /dev
tmpfs                    502M     0  502M    0% /dev/shm
tmpfs                    502M  8.1M  494M    2% /run
tmpfs                    502M     0  502M    0% /sys/fs/cgroup
/dev/sda5                2.0G   33M  2.0G    2% /data/xfs
/dev/sda6                923M  2.4M  857M    1% /data/ext4
/dev/sda2               1014M  204M  811M   21% /boot
/dev/mapper/centos-home  5.0G  1.1G  4.0G   22% /home
tmpfs                    101M  4.0K  101M    1% /run/user/42
tmpfs                    101M   40K  101M    1% /run/user/1000
```

### lsblk 列出系统上所有磁盘的列表
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
根目录是在/dev/sda3，sda5、sda6、sda7是扩展分区sda4上的逻辑分区。</br>
分区4,5,6,7都是我之前练习建的，要删除它们。</br>

## 查看磁盘的分隔方式
现时linux上的磁盘分割方式有两种：GPT和MBR。</br>
MBR的分割方式比GPT要旧一点，MBR分割方式的磁盘大小小于2T，所以现在一般用GPT。</br>
扩容之前一定要查一查分割方式，因为会用到的命令不一样，MBR用的是fdisk，GPT用的是gdisk，如果命令用错了会造成磁盘损坏！！！</br>
</br>
parted 列出磁盘分割表类型和分割资信</br>
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
所以之后命令也就选用fdisk而不是gdisk了。

## 删除多余的没用分区
因为sda5、sda6、sda7是扩展分区sda4上的逻辑分区，所以直接删除4就OK了。

```

#  fdisk /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：M
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

输入p查看分区信息

命令(输入 m 获取帮助)：P

磁盘 /dev/sda：64.4 GB, 64424509440 字节，125829120 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000c9cf3

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1            2048        6143        2048   83  Linux
/dev/sda2   *        6144     2103295     1048576   83  Linux
/dev/sda3         2103296    65026047    31461376   8e  Linux LVM
/dev/sda4        65026048    83886079     9430016    5  Extended
/dev/sda5        65028096    69222399     2097152   83  Linux
/dev/sda6        69224448    71178239      976896   83  Linux
/dev/sda7        71706624    75612159     1952768   83  Linux

命令(输入 m 获取帮助)：d
分区号 (1-7，默认 7)：4
分区 4 已删除

命令(输入 m 获取帮助)：p

磁盘 /dev/sda：64.4 GB, 64424509440 字节，125829120 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000c9cf3

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1            2048        6143        2048   83  Linux
/dev/sda2   *        6144     2103295     1048576   83  Linux
/dev/sda3         2103296    65026047    31461376   8e  Linux LVM

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。
```

w 写入后会要求重启，此处懒得重启，直接使用命令让内核重新找一次分区表：
```
# partprobe 
```
然后，上面警告说sda5和sda6可能正在被使用，要求reboot。</br>
如果确认这两个分区是没有用的，那就reboot一下：</br>
```
# reboot now
```

没有reboot成功，进入了emergency mode。</br>
输入root的密码，进入命令行界面，大概是之前设置了开机挂载，而现在还没有修改/etc/fstab的原因导致的。</br>
```
# cat /etc/fstab
```
查看/etc/fstab文件，用vim删除之前自动挂载的和sda5和sda6有关的两行命令，保存退出，重启。</br>
之后就可以正常进入系统了。</br>
</br>
查看一下多余的分区是否被删除：</br>
```
# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk 
sda               8:0    0   60G  0 disk 
├─sda1            8:1    0    2M  0 part 
├─sda2            8:2    0    1G  0 part /boot
└─sda3            8:3    0   30G  0 part 
  ├─centos-root 253:0    0   10G  0 lvm  /
  ├─centos-swap 253:1    0    1G  0 lvm  [SWAP]
  └─centos-home 253:2    0    5G  0 lvm  /home
sr0              11:0    1 1024M  0 rom  
```

结论：</br>
对系统磁盘修改的同时，一定要记住修改和开机挂载有关的那个文档 /etc/fstab ！
