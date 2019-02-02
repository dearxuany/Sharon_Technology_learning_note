# Python 操作系统相关模块 os
## 目录相关
当前目录
```
[sunnylinux@centOSlearning python3_os]$ tree
.
├── python3_os.py
└── python3_os_test
    ├── test
    │   └── test2
    └── test.txt

3 directories, 2 files


```
os.walk(now_path)返回值包含 ：</br>
(当前目录路径、当前目录所包含的文件及目录的文件名)</br>
(下级目录路径、下级目录所包含的目录和文件名)</br>
(第2层下级目录路径、第2层下级目录所包含的目录和文件名)</br>
http://www.runoob.com/python/os-walk.html
```
#! /usr/bin/env python3

import os

now_path = os.getcwd()  # 当前目录路径

full_dir_list = os.walk(now_path)  # 返回一个含有三个tuple的对象

dir_list = os.listdir(now_path)  


print(now_path)
print(list(full_dir_list))
print(dir_list)
~                  

[sunnylinux@centOSlearning python3_os]$ python3 python3_os.py 
/home/sunnylinux/pythontest/python3_os
[('/home/sunnylinux/pythontest/python3_os', ['python3_os_test'], ['python3_os.py']), ('/home/sunnylinux/pythontest/python3_os/python3_os_test', ['test'], ['test.txt']), ('/home/sunnylinux/pythontest/python3_os/python3_os_test/test', ['test2'], []), ('/home/sunnylinux/pythontest/python3_os/python3_os_test/test/test2', [], [])]
['python3_os_test', 'python3_os.py']


```
