# Linux 文件系统根目录所在磁盘扩容
起因：</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/Linux%20%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%A0%B9%E7%9B%AE%E5%BD%95%E6%89%80%E5%9C%A8%E7%A3%81%E7%9B%98%E6%89%A9%E5%AE%B9.png) </br>
机器是在VMware上搭建的CentOS7，磁盘容量占40GB。</br>
开头以为是物理硬盘不够了，还给虚拟机在VMware上扩大了容量，结果发现根目录所在的磁盘还有一部分位置没有使用，重新把它们分配给根目录用就可以了。</br>
此处主要涉及linux的LVM部分的知识，如果是真的是虚拟机的物理磁盘不够用了，配置会比这个复杂一些，具体还是要查看LVM的资料。</br>

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
可以看到根目录已经用了97%，剩余363M了

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
根目录是在/dev/sda3，sda5、sda6、sda7是扩展分区sda4上的逻辑分区。
（话说，有这么多个分区其实是我很久之前做linux练习时建的分区，4,5,6,7都是我之前练习建的... 已经被我删除了，此处暂不记录）

## 给根目录所在分区扩容
### PV阶段
查看是否有PV存在
```
# pvscan
  PV /dev/sda3   VG centos          lvm2 [30.00 GiB / 14.00 GiB free]
  Total: 1 [30.00 GiB] / in use: 1 [30.00 GiB] / in no VG: 0 [0   ]
```
然后，发现/dev/sda3还有14GB的剩余，还没有VG，所以暂且不需要新增其他物理分区。


### VG阶段
查看现有的VG状态
```
# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               30.00 GiB
  PE Size               4.00 MiB
  Total PE              7680
  Alloc PE / Size       4096 / 16.00 GiB
  Free  PE / Size       3584 / 14.00 GiB
  VG UUID               5lNG9P-lfzt-Btwr-MUu6-a71v-il6q-AaWfY3
```
这里看到已经有一个名为 centos的VG了，然后还有14GB可以使用。


### 划分LV阶段
查看一下已经划分的LV
```
#  lvscan
  ACTIVE            '/dev/centos/swap' [1.00 GiB] inherit
  ACTIVE            '/dev/centos/home' [5.00 GiB] inherit
  ACTIVE            '/dev/centos/root' [10.00 GiB] inherit
```
可以看见，其中一些LV已经分配给了 '/dev/centos/root' [10.00 GiB]，现在只需要直接从VG取空间给这个LV扩大它容量就可以了。</br>
</br>
使用lvextend 增加已有lv容量</br>
```
# lvextend -L +14G /dev/centos/root
  Size of logical volume centos/root changed from 10.00 GiB (2560 extents) to 24.00 GiB (6144 extents).
  Logical volume centos/root successfully resized.
```

再次查看一下
```
#  lvscan
  ACTIVE            '/dev/centos/swap' [1.00 GiB] inherit
  ACTIVE            '/dev/centos/home' [5.00 GiB] inherit
  ACTIVE            '/dev/centos/root' [24.00 GiB] inherit
```
可以看到，现在/dev/centos/root所在的LV已经被扩展到24GB了。


## 刷新文件系统大小
查一下文件系统类型和现时文件系统所占的大小：
```
# df -hT
文件系统                类型      容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root xfs        10G  9.7G  365M   97% /
```
可以看到此时还是只有10GB的，文件系统类型是XFS，要刷新一下大小。</br>
</br>
刷新文件系统所占容量 （ext4用的是resize2fs）</br>
```
# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2621440 to 6291456
```
重新查看一下文件系统所占大小
```
# df -hT
文件系统                类型      容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root xfs        24G  9.7G   15G   41% /
```
至此，扩容就完成了！！

