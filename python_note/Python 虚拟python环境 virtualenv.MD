# Python 虚拟python环境 virtualenv
virtualenv 可以搭建任意多个小型、独立的“虚拟的python环境”，以便保护系统环境和管理python的各种包，如果不需要某些包，只需删除对应的虚拟环境目录即可。

## linux 上 virtualenv 的安装
下载地址：https://pypi.org/project/virtualenv/ </br>
注意：需要root权限</br>

进入python3所在文件夹
```
# cd /usr/local/python3
```
用pip3安装virtualenv
```
# pip install virtualenv
Installing collected packages: virtualenv
  The script virtualenv is installed in '/usr/local/python3/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed virtualenv-16.0.0
```

## virtualenv 的使用
进入到要新建虚拟环境的目录</br>
新建一个虚拟python环境目录</br>
目录名可自定义，之后虚拟环境会被创建在此目录中</br>
```
$ virtualenv geo_env
bash: virtualenv: 未找到命令...
```
大概是因为系统中有python2和3两个版本，所以要在系统中建立软链</br>
</br>
确定python3安装位置</br>
```
$ whereis python3
python3: /usr/bin/python3 /usr/local/python3
```
virtualenv软链到系统bin里面
```
$ sudo ln -s /usr/local/python3/bin/virtualenv /usr/bin/virtualenv
```
之后就可以使用命令virtualenv 创建虚拟环境了
```
$ virtualenv -p python3 geo_env
Already using interpreter /usr/bin/python3
```
注意：</br>
默认情况下，虚拟环境会依赖系统环境中的site packages，就是说系统中已经安装好的第三方package也会被复制到虚拟环境中，如果不想依赖这些package，那么可以加上参数 --no-site-packages。</br>
</br>
因为上面一条在虚拟环境下复制setuptool时好点问题，尝试不装site-packages里面的包</br>
```
$ virtualenv --no-site-packages geo_env
```
还是出现了以下错误
```
OSError: Command /home/sunnylinux/pyt.../geo_env/bin/python3 - setuptools pip wheel failed with error code 2
```
貌似是setuptool和pip版本太久的原因，尝试更新到最新版本
```
$ sudo pip3 install --upgrade --default-timeout=100 setuptools
Installing collected packages: setuptools
  Found existing installation: setuptools 28.8.0
    Uninstalling setuptools-28.8.0:
      Successfully uninstalled setuptools-28.8.0
Successfully installed setuptools-40.4.3
$ sudo pip3 install --upgrade virtualenv
Successfully installed virtualenv-16.1.0
```
尝试安装pysocks
```
$ sudo pip3 install pysocks
$ unset all_proxy
```
重新创建虚拟环境geo_env
```
$ virtualenv -p python3 geo_env
Already using interpreter /usr/bin/python3
Using base prefix '/usr/local/python3'
New python executable in /home/sunnylinux/pythontest/python3_env/geo_env/bin/python3
Also creating executable in /home/sunnylinux/pythontest/python3_env/geo_env/bin/python
Installing setuptools, pip, wheel...
done.
```
进入虚拟环境所在文件夹并激活虚拟环境
```
$ cd ./geo_env/
$ ls
bin  include  lib
$ source ./bin/activate
(geo_env) [sunnylinux@centOSlearning geo_env]$
```
注意：括号中的内容表示在虚拟环境内
</br>
在虚拟环境中安装各种第三方包
```
$ pip3 install pygeocoder
Successfully installed certifi-2018.10.15 chardet-3.0.4 idna-2.7 pygeocoder-1.2.5 requests-2.20.0 urllib3-1.24.1
$ python3 -c 'import pygeocoder'
```
退出当前虚拟环境
```
(geo_env) [sunnylinux@centOSlearning geo_env]$ deactivate
[sunnylinux@centOSlearning geo_env]$ 
```
