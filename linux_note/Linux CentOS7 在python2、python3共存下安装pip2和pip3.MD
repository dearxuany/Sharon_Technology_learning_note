# Linux CentOS7 在python2、python3共存状态下安装pip2和pip3
## 安装驱动
### pip一般安装方法
pip 是一个通用的 Python 包管理工具，提供了对 Python 包的查找、下载、安装、卸载的功能。</br>
官方教程 https://pip.pypa.io/en/stable/installing/</br>
pip下载地址 https://pypi.org/project/pip/</br>
</br>
root下运行以下命令
```
[root@centOSlearning python3]# wget https://bootstrap.pypa.io/get-pip.py
[root@centOSlearning python3]# python3 get-pip.py
```
```
#可以看到以下输出
Successfully installed pip-18.0 wheel-0.31.1
[root@centOSlearning python3]# pip -V
pip 18.0 from /usr/lib/python2.7/site-packages/pip (python 2.7)		#这里发现装给了2.7
```

## pip2和pip3共存的安装方法
### 创建软链接
创建Pip3的软链接
```
# ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```
设置环境变量
```
# PATH=$PATH:/usr/local/python3/bin/pip3
# export PATH
```
执行行下面的命令，让上一步的修改生效
```
# source ~/.bash_profile
```
结果：没有解决

### 下载pip3
下载pip3安装包（有些时候会很慢...）
```
# wget https://files.pythonhosted.org/packages/69/81/52b68d0a4de760a2f1979b0931ba7889202f302072cc7a0d614211bc7579/pip-18.0.tar.gz
```
解压
```
# tar -zxvf pip-18.0.tar.gz
```
进入pip所在路径的目录
```
# cd pip-18.0
# python3 setup.py build
Traceback (most recent call last):
  File "setup.py", line 6, in <module>
    from setuptools import find_packages, setup
ModuleNotFoundError: No module named 'setuptools'
```
报错显示没有setuptools，去下载一个，地址：https://pypi.org/project/setuptools/
```
# wget https://files.pythonhosted.org/packages/d3/3e/1d74cdcb393b68ab9ee18d78c11ae6df8447099f55fe86ee842f9c5b166c/setuptools-40.0.0.zip
```
解压
```
# unzip setuptools-40.0.0.zip
```
进入setuptools的目录
```
# cd setuptools-40.0.0/
```
安装
```
# python3 setup.py build
ModuleNotFoundError: No module named '_ctypes'
```
貌似是3.7版本需要一个新的包libffi-devel，安装一下
```
# yum install libffi-devel -y
```
安装好新模块后，重新进入python 3.7的安装目录，一定要重新编译一下！
```
# pwd
/usr/local/python3/Python-3.7.0a1
# make && make install
```
然后最后看到这几行：
```
Collecting setuptools
Collecting pip
Installing collected packages: setuptools, pip
Successfully installed pip-9.0.1 setuptools-28.8.0
```
## 查看pip版本信息
查看一下pip的版本信息
```
# pip3 -V
pip 9.0.1 from /usr/local/python3/lib/python3.7/site-packages (python 3.7)
# pip2 -V
pip 18.0 from /usr/lib/python2.7/site-packages/pip (python 2.7)
```

## 更新pip3
进入到之前下好和解压好的pip-18.0目录
```
# cd pip-18.0
# python3 setup.py build
# python3 setup.py install
```
更新好后，查看一下版本号
```
# pip3 -V
pip 18.0 from /usr/local/python3/lib/python3.7/site-packages/pip-18.0-py3.7.egg/pip (python 3.7)
```

结论：</br>
貌似是由于缺少libffi-devel模块的原因，才导致一开始安装3.7的时候没有配套的pip3。</br>
安装好模块后，重新编译一下就好了。如果想用更新版本的pip3也可以自行安装。</br>
