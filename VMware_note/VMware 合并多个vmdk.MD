# 合并多个vmdk
## 找Vmware Station 自带的命令行工具"vmware-vdiskmanager"记住其路径
D:\VMware 9_i386

## 找要合并的虚拟机的VMDK存放位置和名字
D:\CentOS\Clone of CentOS7 40GB
CentOS7 learning 40GB-cl1.vmdk

## 打开cmd执行以下命令
vmware-vdiskmanager.exe -r "待转换vmdk路径" -t 0 "新生成的vmdk文件路径"</br>
进入D盘
```
D:
```
进入当前盘"vmware-vdiskmanager"所在目录
```
cd ./VMware 9_i386 
```
输入以下命令，新生vmdk路径不填的话默认和原vmdk一样
```
vmware-vdiskmanager.exe -r "D:\CentOS\Clone of CentOS7 40GB\CentOS7 learning 40GB-cl1.vmdk" -t 0 "C:\Users\Tommy\Desktop\VMware虚拟机备份文件\MyNewImage.vmdk"
```
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/VMware_note_images/VMware%20%E5%90%88%E5%B9%B6%E5%A4%9A%E4%B8%AAvmdk2.png)</br>
（其实路径最后也不要有中文，很有可能会出现乱码或者其他问题。这里虽然乱码了，但合并还是成功的。）


## 检验测试
合并完毕后，可利用合并后的vmdk新建一个虚拟机，也可覆盖原来的那些vmdk并继续使用原来的虚拟机，不过要改成和原有虚拟机vmdk一样的名字，以及放在和原vmdk同样的位置。</br>
建议是利用合并后的vmdk先新建一个主机看看vmdk是否合并正常，检查虚拟机是否是需要的状态。经检验此次合并的内容是不包含子快照的数据的。</br>
