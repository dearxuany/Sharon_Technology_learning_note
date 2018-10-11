# Python 程序打包 Setuptools
Setuptools 工具包可用于编写安装脚本，还可用于编译扩展。</br>
按约定，安装脚本被命名为setup.py ，该脚本可用于安装模块、包、扩展。</br>
Setuptools的安装：</br>
有pip可用pip装，没有用wget到https://pypi.org/project/setuptools/ 下一个</br>
详情见 [Linux CentOS7 在python2、python3共存状态下安装pip2和pip3](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20CentOS7%20%E5%9C%A8python2%E3%80%81python3%E5%85%B1%E5%AD%98%E4%B8%8B%E5%AE%89%E8%A3%85pip2%E5%92%8Cpip3.MD#%E4%B8%8B%E8%BD%BDpip3)

## 创建setup.py 安装脚本
要把一个具有快排的小程序打包成安装包，打包使用setuptools模块中的setup</br>
注意：</br>
新建安装包前要注意当前目录拥有要打包的源码文件；</br>
要注意当前目录位置，因为新建的安装脚本会覆盖源码文件，所以注意新建存储安装包的目录和备份。</br>

```
#! /usr/bin/python3

from setuptools import setup

setup(
    name='python3_algorithms_Quicksort',  # 应用名
    version='1.0',  # 版本
    description='Quick Sort',  # 描述
    author='Sharon Li',
    py_modules=['python3_algorithms_Quicksort']  #  重要：要打包的文件名 
    )
~          
```
命令行运行上面的脚本，此是setuptools其实还没开始运行，这样运行会介绍参数的用法
```
$ python3 setup.py
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help

error: no commands supplied
```
使用build运行setuptools生成一个名为build的目录，其中包含lib，要打包的代码会被复制到lib中
```
$ python3 setup.py build
running build
running build_py
creating build
creating build/lib
copying python3_algorithms_Quicksort.py -> build/lib
```
```
$ ls -a
.  ..  build  python3_algorithms_Quicksort.py  setup.py
```
## 安装
使用install 安装这个模块，注意安装需要root权限
```
# python3 setup.py install
running install
running bdist_egg
running egg_info
creating python3_algorithms_Quicksort.egg-info
（省略部分输出）
Installed /usr/local/python3/lib/python3.7/site-packages/python3_algorithms_Quicksort-1.0-py3.7.egg
Processing dependencies for python3-algorithms-Quicksort==1.0
Finished processing dependencies for python3-algorithms-Quicksort==1.0
```
该命令会将当前的Python应用安装到当前Python环境的”site-packages”目录下，这样其他程序就可以像导入标准库一样导入该应用的代码了。
注意：貌似是没有标准卸载命令的，所以要卸载的话需要手动卸载，要注意系统环境保护！
```
>>> from python3_algorithms_Quicksort import quicksort
>>> quicksort([3,7,834,35,74,767,333,565,7443,88568,846835,34636])
[3, 7, 35, 74, 333, 565, 767, 834, 7443, 34636, 88568, 846835]
>>> 
```
开发式安装	
```
# python setup.py develop
```
使用”develop”开发方式安装的话，应用代码不会真的被拷贝到本地Python环境的”site-packages”目录下，而是在”site-packages”目录里创建一个指向当前应用位置的链接。这样如果当前位置的源码被改动，就会马上反映到”site-packages”里，以防止系统环境被破坏。


## 压缩打包
setuptools 还可用于穿件源代码归档文件，可将安装包打包成 *.tar.gz文件(sdist)、Win安装程序(py2exe)、Mac安装程序(py2app)、RPM包、egg文件(bdist_egg)、wheel文件等，还可使用参数--formats一次打包成多种不同分发格式，以便分发给其他人使用。

### 打包成*.tar.gz文件
```
# python3 setup.py sdist
```
命令会生成一个dist目录，里面包含了源码和它的*.tar.gz文件

### 打包成egg文件
```
# python setup.py bdist_egg
```
命令会生成一个以*.egg-info 为结尾的文件名，里面包含以下一些东西
```
# ls -a
.  ..  dependency_links.txt  PKG-INFO  SOURCES.txt  top_level.txt
```
