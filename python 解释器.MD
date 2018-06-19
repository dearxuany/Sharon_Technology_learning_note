# 关于python解释器
Python 解释器可不止一种，有 CPython、IPython、Jython、PyPy 等。<br>
CPython 就是用 C 语言开发的了，是官方标准实现，拥有良好的生态，所以应用也就最为广泛了。<br>
IPython 是在 CPython 的基础之上在交互式方面得到增强的解释器（http://ipython.org ）。 <br>
Jython 是专为 Java 平台设计的 Python 解释器（http://www.jython.org ），它把 Python 代码编译成 Java 字节码执行。<br>
PyPy 是 Python 语言（2.7.13和3.5.3）的一种快速、兼容的替代实现（http://pypy.org ），以速度快著称。<br>

# 脚本式编程
* 后缀<br>
\*.py   
linux系统下，文件的后缀对计算机来说是没有特殊意义的，会和其他字符一起被当做文件名一部分来处理，仅用作给人提供辨识。

* 声明
```
#! /usr/bin/env python3
```

实验脚本
```
#! /usr/bin/env python3
# Program:
#	This program is used to test the use of python3 script.
# History:
# 	2018/05/18 Sharon 1st

x=input("place input the value of x: ")
y=input("place input the value of y: ")
print(x,y)
```

* 执行脚本
```
python3 script_name.py
```
