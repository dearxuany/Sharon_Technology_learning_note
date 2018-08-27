# Python 模块
模块：</br>
如果直接在python解释器中编程而不保存，当从python解释器退出再进入，那么先前定义的函数和变量就都会被删除不能再使用了，于是有了模块。模块是包含所有定义的类、函数和变量的文件，其后缀名是.py。模块可以被别的程序引入，以使用该模块中的函数等功能，可让相同代码得到复用而不用重新编写。

## 模块定义
### 在模块中定义函数
某个名为 python3_fibs.py 的python小程序，用于计算不同位数的斐波那契数列（Fibonacci sequence）数列：
```
#!/usr/bin/python2

def fibs():
    'Fibonacci sequence'
    num=int(input('How many Fibonacci number do you want? '))
    Fib_seq=[0,1]
    for n in range(num-2):
        Fib_seq.append(Fib_seq[-2]+Fib_seq[-1])
    return print(Fib_seq)

# 调用函数fibs，作为模块导入时要删除此句
fibs()  
```
在bash中直接调用执行 *.py
```
$ python3 python3_fibs.py 
How many Fibonacci number do you want? 16
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610]
```
将上述 *.py 修改为适用于作为模块的程序：</br>
模块仅包含定义的类、函数和变量，不应包含执行这些函数的代码段。</br>
```
#!/usr/bin/python2

def fibs(num):
    'Fibonacci sequence'
    Fib_seq=[0,1]
    for n in range(num-2):
        Fib_seq.append(Fib_seq[-2]+Fib_seq[-1])
    return print(Fib_seq)
```

### 让模块可用
在python3解释器将 *.py 程序作为模块导入</br>
让解释器正常导入模块有以下两种方法：</br>
1.找到解释器查找模块的原始路径，将新的模块放在这个位置，这样做需要管理员权限；</br>
2.告诉解释器到正确的新路径去找模块，将模块所在目录包含在环境变量PYTHONPATH中。</br>
参照：python打包用户指南 https://packaging.python.org/ </br>
</br>
在模块sys中查看模块放置路径：
```
>>> import sys
>>> import pprint	# 分行的打印函数
>>> pprint.pprint(sys.path)
['',
 '/usr/local/python3/lib/python37.zip',
 '/usr/local/python3/lib/python3.7',
 '/usr/local/python3/lib/python3.7/lib-dynload',
 '/usr/local/python3/lib/python3.7/site-packages',
 '/usr/local/python3/lib/python3.7/site-packages/pip-18.0-py3.7.egg']
```
一般来说，放在site-packages最好，但是最好还是自己新建一个目录，让解释器去找。</br>
</br>
在bash中，将模块所在目录添加在环境变量PYTHONPATH中：</br>
```
# 环境变量声明
$ export PYTHONPATH=$PYTHONPATH:~/home/sunnylinux/pythontest/python3_module
# 查看环境变量
$ echo $PYTHONPATH
:/home/sunnylinux/home/sunnylinux/pythontest/python3_module
```
在python3解释器中导入模块：
```
>>> import python3_fibs
>>> python3_fibs.fibs(15)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
```
### 在模块中添加测试代码
使用__name__来显示当前位置（主程序？模块？）：</br>
如果当前程序是主程序，那么 \_\_name\_\_ 会变成 ’\_\_main\_\_’ ；</br>
如果当前程序是被主程序导入的模块，那么 \_\_name\_\_ 会变成当前模块的模块名。</br>
```
# python3_fibs作为模块被导入到python3解释器中
>>> __name__
'__main__'
>>> python3_fibs.__name__
'python3_fibs'
```
有时想检查模块中的代码是否可用，可是模块中不能有执行其定义函数的代码，则让模块难以得到测试。可在模块中添加测试代码以对模块进行测试，即仅当\_\_name\_\_为\_\_main\_\_时执行测试代码。</br>
</br>
修改python3_fibs模块：</br>
```
#!/usr/bin/python2

def fibs(num):
    'Fibonacci sequence'
    Fib_seq=[0,1]
    for n in range(num-2):
        Fib_seq.append(Fib_seq[-2]+Fib_seq[-1])
    return print(Fib_seq)

if __name__ == '__main__':
    num=int(input('How many Fibonacci number do you want? '))
    fibs(num)
```
在bash中直接执行python3_fibs.py ：
```
$ python3 python3_fibs.py 
How many Fibonacci number do you want? 9
[0, 1, 1, 2, 3, 5, 8, 13, 21]
```
在python3解释器中导入该模块：
```
>>> import python3_fibs
>>> python3_fibs.fibs(22)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946]
>>> 
```

## 模块内容查询
### 函数 dir()
dir(module_name) ：列出对象的所有属性。对于模块来说，可列出模块包含的函数、类、变量等。
```
>>> dir(python3_fibs)
['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'fibs']
```
上面有前后有下划线的名称表示改函数/属性无法供外部使用，没有下划线开头的则是可被外部调用的。</br>
如果这些变量很多的情况下，可使用列表推导进行过滤：</br>
```
>>> [n for n in dir(python3_fibs) if not n.startswith('_')]
['fibs']
```
### 仅导入模块中某些变量 
变量__all__ ：</br>
模块中变量 __all__ 中显示的内容，为可以使用 from module_name import sub_module_name 形式单独导入的模块。如果不设置 __all__ ，则使用 import * 形式导入所有不以下划线打头的全局变量。
```
>>> dir(copy)
['Error', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_copy_dispatch', '_copy_immutable', '_deepcopy_atomic', '_deepcopy_dict', '_deepcopy_dispatch', '_deepcopy_list', '_deepcopy_method', '_deepcopy_tuple', '_keep_alive', '_reconstruct', 'copy', 'deepcopy', 'dispatch_table', 'error']
>>> copy.__all__
['Error', 'copy', 'deepcopy']
```
在下次需要导入copy模块时，则可选择使用 from copy import copy 仅仅导入copy ，而不导入Error和deepcopy。
```
>>> x
3.141592653589793
>>> from copy import copy
>>> y=copy(x)
>>> y
3.141592653589793
>>> z=copy.copy(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'function' object has no attribute 'copy'
>>> k=deepcopy(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'deepcopy' is not defined
```

像math模块这样没有__all__的也可使用 from module_name import sub_module_name 来导入子模块，不过要注意调用时的方法不同：
```
>>> dir(math)
['__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 'atanh', 'ceil', 'copysign', 'cos', 'cosh', 'degrees', 'e', 'erf', 'erfc', 'exp', 'expm1', 'fabs', 'factorial', 'floor', 'fmod', 'frexp', 'fsum', 'gamma', 'gcd', 'hypot', 'inf', 'isclose', 'isfinite', 'isinf', 'isnan', 'ldexp', 'lgamma', 'log', 'log10', 'log1p', 'log2', 'modf', 'nan', 'pi', 'pow', 'radians', 'remainder', 'sin', 'sinh', 'sqrt', 'tan', 'tanh', 'tau', 'trunc']

# 重启解释器
>>> from math import sqrt
>>> math.sqrt(4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'math' is not defined
>>> sqrt(4)
2.0
```
### 帮助文档
查看有关模块的帮助信息有以下两种方法：</br>
帮助函数 help(module_name)和 文档字符串module_name.\_\_doc\_\_ ，help()下的帮助信息会比__doc__下的多。</br>
</br>
想仅查看模块内某函数的说明，可直接查函数的__doc__：</br>
```
>>> print(sqrt.__doc__)
Return the square root of x.
```
### 查看源码
查看模块源码位置
```
>>> print(math.__file__)
/usr/local/python3/lib/python3.7/lib-dynload/math.cpython-37m-i386-linux-gnu.so
>>> print(copy.__file__)
/usr/local/python3/lib/python3.7/copy.py
```
