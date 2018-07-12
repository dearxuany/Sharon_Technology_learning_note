# VMware 物理机磁盘空间不足的问题解决
## 清理磁盘
VM - manage - clear up disks</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20%E7%89%A9%E7%90%86%E6%9C%BA%E7%A3%81%E7%9B%98%E7%A9%BA%E9%97%B4%E4%B8%8D%E8%B6%B3.png)</br>
（此处显示不用清理，未能解决我的问题）

## 重启物理机
重启后可以正常开启虚拟机，但是运行一段时间后会重现以上问题。</br>


## 增加物理分区容量
物理机 压缩卷-扩展卷</br>
其余磁盘能压缩的空间不多，不足以满足需求。

## 合并多个vmdk
命令行操作，个人觉得也挺方便。</br>
不过合成后的vmdk不会包含child snapshot，释放的空间可能不多。</br>
具体操作查看：[VMware 合并多个vmdk](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/VMware_note/VMware%20%E5%90%88%E5%B9%B6%E5%A4%9A%E4%B8%AAvmdk.MD)

## VMware删除多余快照
正常删除的情况下，能有效释放大量空间。</br>
可是操作间容易出现错误，容易造成数据丢失。</br>
具体操作查看：[VMware 删除旧的快照](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/VMware_note/VMware%20%E5%88%A0%E9%99%A4%E6%97%A7%E7%9A%84%E5%BF%AB%E7%85%A7.MD)

## full clone 虚拟机
完整克隆虚拟机为一台新的计算机，将所有快照合成为一个，正常运行后，删除原来的虚拟机及其文件夹。</br>
