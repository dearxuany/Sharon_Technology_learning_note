# Percona XtraBackup 安装
## 下载安装包
官网地址： https://www.percona.com/doc/percona-xtrabackup/2.3/installation.html?spm=a2c4g.11186623.2.12.22af2374UdROqg </br>
</br>
安装依赖</br>
```
# yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
Loaded plugins: fastestmirror
percona-release-0.1-4.noarch.rpm                                | 6.4 kB  00:00:00     
Examining /var/tmp/yum-root-x6ec8R/percona-release-0.1-4.noarch.rpm: percona-release-0.1-4.noarch
Marking /var/tmp/yum-root-x6ec8R/percona-release-0.1-4.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package percona-release.noarch 0:0.1-4 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================
 Package              Arch        Version     Repository                          Size
=======================================================================================
Installing:
 percona-release      noarch      0.1-4       /percona-release-0.1-4.noarch      5.8 k

Transaction Summary
=======================================================================================
Install  1 Package

Total size: 5.8 k
Installed size: 5.8 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : percona-release-0.1-4.noarch                                        1/1 
  Verifying  : percona-release-0.1-4.noarch                                        1/1 

Installed:
  percona-release.noarch 0:0.1-4                                                       

Complete!
```
检验目录是否可用
```
yum list | grep percona
```
安装  percona-xtrabackup
```
yum install percona-xtrabackup
```
