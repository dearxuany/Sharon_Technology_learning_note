# Python *.py直接执行和导入解释器执行的传参问题
## 问题描述
有这样一个程序
```
#! /usr/bin/env python3

def func2(c):
    print('func2')
    print('a=',a)
    print('b=',b)
    print(c)

def func1(a,b):
    print('func1')
    c=3
    print('a=',a)
    print('b=',b)
    func2(c)

if __name__ == '__main__':
   a = 1
   b = 2

   func1(a,b)
```
直接作为脚本执行，它是运行正常的：
```
$ python3 func_test.py
func1
a= 1
b= 2
func2
a= 1
b= 2
3
```
但是导入到python3解释器，把它作为模块使用就会出现这样的问题
```
>>> import func_test
>>> a=1
>>> b=2
>>> func_test.func1(a,b)
func1
a= 1
b= 2
func2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/sunnylinux/pythontest/python3_script/func_test.py", line 14, in func1
    func2(c)
  File "/home/sunnylinux/pythontest/python3_script/func_test.py", line 5, in func2
    print('a=',a)
NameError: name 'a' is not defined

>>> func1(1,2)
func1
a= 1
b= 2
func2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/sunnylinux/pythontest/python3_script/func_test.py", line 14, in func1
    func2(c)
  File "/home/sunnylinux/pythontest/python3_script/func_test.py", line 5, in func2
    print('a=',a)
NameError: name 'a' is not defined
```

## 问题解决
```
#! /usr/bin/env python3

def func2(c):
    print('func2')
    print('a=',a)
    print('b=',b)
    print(c)

def func1(x,y):
    global a
    global b

    a=x
    b=y

    print('func1')
    c=3
    print('a=',a)
    print('b=',b)
    func2(c)

if __name__ == '__main__':
   a = 1
   b = 2
   
   func1(a,b)
```
```
$ python3 func_test.py
func1
a= 1
b= 2
func2
a= 1
b= 2
3
```
```
$ python3
Python 3.7.0a1 (default, May 18 2018, 14:10:18) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from func_test import *
>>> a=1
>>> b=2
>>> func1(a,b)
func1
a= 1
b= 2
func2
a= 1
b= 2
3

>>> func1(3,4)
func1
a= 3
b= 4
func2
a= 3
b= 4
3

```
