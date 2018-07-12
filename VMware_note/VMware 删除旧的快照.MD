# VMware 删除旧的快照
## 备份vmdk
为避免删除过程中出现问题，最好先完整复制一份到其他存储位置，防止因不正常删除快照而造成数据丢失。
## 打开Snapshot Manager
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20%E5%88%A0%E9%99%A4%E6%97%A7%E7%9A%84%E5%BF%AB%E7%85%A7.png)
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20%E5%88%A0%E9%99%A4%E6%97%A7%E7%9A%84%E5%BF%AB%E7%85%A72.png)
## 选择需要删除的快照后，按Delete，开始压缩并删除快照
过程中最好不要执行其他操作，耐心等待。否则，如果在删除中途出现问题，可能导致数据缺失、虚拟机损坏难以恢复。通常，删除成功后物理硬盘会多出几个G的空间。
