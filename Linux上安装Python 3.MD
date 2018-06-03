# 在Linux上安装Python3
## 安装依赖环境
需要root权限<br>
```
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```
## 到python官网找到下载路径, 用wget下载
https://www.python.org/ftp/python/
此处下载python 3.7.0<br>
```
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0a1.tgz
```

## 解压后，在/usr/local/python3下安装Python3
解压tgz包
```
tar -zxvf Python-3.7.0a1.tgz
```
新建安装目录 /usr/local/python3
```
cd /usr
cd local
mkdir python3
```
移动安装包到/usr/local/python3，注意权限
```
mv ./Python-3.7.0a1 /usr/local/python3/
```
进入解压后的目录
```
cd Python-3.7.0a1
```
添加配置
```
./configure --prefix=/usr/local/python3
```
配置自己的安装目录，编译源码
```
make
```
执行安装
```
make install
```
## 建立python3的软链
系统中原来的python在/usr/bin/python，通过ls -l可以看到，python是一个软链接，链接到本目录下的python2.6。
安装后系统使用的python还是默认的python2.x，需修改为python3.x。
新建一个python3的软链接，不要删掉python2的，不然以前版本的会运行不了。
```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```
并将/usr/local/python3/bin加入PATH
```
PATH=$PATH:/usr/local/python3/bin/python3    # 设置环境变量
export PATH
```
执行行下面的命令，让上一步的修改生效
```
source ~/.bash_profile
```

## 检查python3是否正常使用
```
python3 -V
```
Python 3.7.0a1


## 启动Python解释器
### 启动python3
```
python3
```
Python 3.7.0a1 (default, May 18 2018, 14:10:18) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
### 启动默认版本python
```
python
```
Python 2.7.5 (default, Apr 11 2018, 08:07:30) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
