# Python 错误和异常
Python中，造成程序不能正常运行的原因有两种：语法错误和异常。</br>
语法错误：程序不符合Python语法，通常会指示最先找到错误的位置并显示SyntaxError: invalid syntax。</br>
异常：发生了通常不会发生的事情，通常是需要人为定义处理方法去解决的实际问题，Python中有很多内置异常类。</br>

## 内置异常
3.7 版本的Python内置异常查询（关键字 Built-in Exceptions）</br>
https://docs.python.org/3.7/library/exceptions.html?highlight=built%20exception</br>
以下为一些常见的异常类</br>

### exception Exception
Exception：通用异常，所有异常类均由此派生。语句raise可引发，还可添加错误信息。</br>
All built-in, non-system-exiting exceptions are derived from this class. All user-defined exceptions should also be derived from this class.
```
>>> raise Exception                   # raise引发通用错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception
>>> raise Exception('hello world!')   # 添加错误信息
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception: hello world!               # 错误信息显示
>>> 
```

### exception AttributeError
AttributeError：引用或给属性赋值失败时引发。</br>
Raised when an attribute reference (see Attribute references) or assignment fails. (When an object does not support attribute references or attribute assignments at all, TypeError is raised.)
```
class Commodity:
    def __init__(self,name):
        self.__name=name

    def sell(self):
        print(self.__name.title(),'is on sell.')

commodity1=Commodity('milk')	
commodity2=Commodity('cake')

commodity2.sell()           # 此句没有出错，实例属性是可以正常被调用的
print(commodity1.__name)    # 出错的是这句，__ 禁止访问属性，删除此句程序会运行正常

# 输出结果
Traceback (most recent call last):
  File "python_u7_object.py", line 65, in <module>
    print(commodity1.__name)
AttributeError: 'Commodity' object has no attribute '__name
```
### exception OSError([arg])
### exception OSError(errno, strerror[, filename[, winerror[, filename2]]])
OSError：操作系统不能执行指定任务时引发，有多个子类。</br>
This exception is raised when a system function returns a system-related error, including I/O failures such as “file not found” or “disk full” (not for illegal argument types or other incidental errors).
The second form of the constructor sets the corresponding attributes, described below. The attributes default to None if not specified. For backwards compatibility, if three arguments are passed, the args attribute contains only a 2-tuple of the first two constructor arguments.
The constructor often actually returns a subclass of OSError, as described in OS exceptions below. The particular subclass depends on the final errno value. This behaviour only occurs when constructing OSError directly or via an alias, and is not inherited when subclassing.

### exception IndexError
IndexError：使用序列中不存在索引时引发，LookupError的子类。</br>
Raised when a sequence subscript is out of range. (Slice indices are silently truncated to fall in the allowed range; if an index is not an integer, TypeError is raised.)
```
>>> list1=[1,2,3,4,5]
>>> list1[7]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> 
```

### exception KeyError
KeyError：使用字典中不存在的key时引发，LookupError的子类。</br>
Raised when a mapping (dictionary) key is not found in the set of existing keys.
```
>>> {'a':1,'b':2}
{'a': 1, 'b': 2}
>>> dict1={'a':1,'b':2}
>>> dict1['a']
1
>>> dict1['c']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'c'
>>> 
```

### exception NameError
NameError：使用没有指向的变量时引发。</br>
Raised when a local or global name is not found. This applies only to unqualified names. The associated value is an error message that includes the name that could not be found.
```
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
>>> 
```
### exception SyntaxError
SyntaxError：代码语法错误。</br>
Raised when the parser encounters a syntax error. This may occur in an import statement, in a call to the built-in functions exec() or eval(), or when reading the initial script or standard input (also interactively).
Instances of this class have attributes filename, lineno, offset and text for easier access to the details. str() of the exception instance returns only the message.
```
>>> def function()
  File "<stdin>", line 1
    def function()
                 ^
SyntaxError: invalid syntax
>>> 
```
### exception TypeError
TypeError：对象类型不正确。</br>
Raised when an operation or function is applied to an object of inappropriate type. The associated value is a string giving details about the type mismatch.
This exception may be raised by user code to indicate that an attempted operation on an object is not supported, and is not meant to be. If an object is meant to support a given operation but has not yet provided an implementation, NotImplementedError is the proper exception to raise.
Passing arguments of the wrong type (e.g. passing a list when an int is expected) should result in a TypeError, but passing arguments with the wrong value (e.g. a number outside expected boundaries) should result in a ValueError.

```
>>> b=input('place input a number:')
place input a number:6
>>> 1+b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'str'
>>> 1+int(b)
7
>>> 
```
### exception ValueError
ValueError：调用对象时，类型正确但包含的值不适合。</br>
Raised when a built-in operation or function receives an argument that has the right type but an inappropriate value, and the situation is not described by a more precise exception such as IndexError.
```
>>> int(z)/3				# z为空
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: ''
```
### exception ZeroDivisionError
ZeroDivisionError：除法、求余，第二个参数为0时引发。</br>
Raised when the second argument of a division or modulo operation is zero. The associated value is a string indicating the type of the operands and the operation.
```
>>> 1/2
0.5
>>> 1/0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>> 1%7
1
>>> 1%0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: integer division or modulo by zero
>>> 
```

## 自定义异常类
可自定义异常类，但必须继承Exception，可以添加方法，创建自定义异常类和创建普通类方法一样。</br>
```
>>> class SomeException(Exception):             # 创建异常类
...     def doSomething(self):
...         return 1+1
... 

>>> raise SomeException                         # 引发新建的异常类
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
__main__.SomeException
```
```
>>> raise SomeException('This is a new errot.') # 给新的异常类加错误信息
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
__main__.SomeException: This is a new errot.
```
```
>>> SomeException.doSomething()                 #  SomeException还没有实例化，所以会出现错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: doSomething() missing 1 required positional argument: 'self'
>>> Error1=SomeException()                      # 实例化，也可以参考抽象基类
>>> Error1.doSomething()
2
```


## 捕获异常
可用try/except捕获异常，可以捕获他人写的异常，不让异常继续向上传播，不用写多条if语句。</br>
使用try/except后，发生异常时，可以仅是给出错误提示，而不对异常做任何处理，不终止程序的运行，让后续的语句得到运行的机会。</br>
try语句按照如下方式工作；</br>
1)执行try子句（在关键字try和关键字except之间的语句）</br>
2)如果没有异常发生，忽略except子句，try子句执行后结束。</br>
3)如果在执行try子句的过程中发生了异常，那么try子句余下的部分将被忽略。</br>
4)如果异常的类型和 except 之后的名称相符，那么对应的except子句将被执行。</br>
5)最后执行 try 语句之后的代码。</br>
6)如果一个异常没有与任何的except匹配，那么这个异常将会传递给上层的try中。</br>
一个 try 语句可能包含多个except子句，分别来处理不同的特定的异常。最多只有一个分支会被执行。</br>
处理程序将只针对对应的try子句中的异常进行处理，而不是其他的 try 的处理程序中的异常。</br>

对比以下连两个例子：</br>
```
# 例子1：无捕获异常
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

div()
print('hello world!')

# 例子1正常输出
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
hello world!

# 例子1出现异常，程序执行被中断，最后的print没有被执行
Place input the first num: 6 
Place input the second num: 0
Traceback (most recent call last):
  File "python3_u8_error.py", line 15, in <module>
    div()
  File "python3_u8_error.py", line 11, in div
    x/y
ZeroDivisionError: division by zero
```
```
# 例子2：有捕获异常

def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()
    print(c)
except (ZeroDivisionError,ValueError):
    print('The input must be a number and the second number cannot be zero.')
except Exception:
    print('other errors')

print('Hello world!')
~                             

#例子2正常输出
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
2.0
Hello world!

# 例子2异常出现，try/except捕获了异常，没有使程序中断，让后面的语句得到了执行
Place input the first num: 6 
Place input the second num: 0
The input must be a number and the second number cannot be zero.		# 出现异常
Hello world!  # 继续往后执行了后面的代码
```



### 捕捉单个异常
```
>>> try:
...     pass
... except ZeroDivisionError:
...     print('The second number cannot be zero.')


try:
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    print('x/y=',x/y)
except ZeroDivisionError:	# 捕捉到异常时错误提示
print('The second number connot be zero.')		

# 第一次执行
Place input the first num: 2
Place input the second num: 6
x/y= 0.3333333333333333
# 第二次执行
Place input the first num: 6
Place input the second num: 0
The second number connot be zero.
```

### 捕捉多个异常
#### 使用多条except语句
多个except子句可以捕捉多种异常，也可以用Exception来代表其他所有类的异常。</br>
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()
    print(c)
except ZeroDivisionError:     # 捕捉除数为0引发的异常
print('The second number connot be zero.')	  
except Exception:             # 捕捉其他异常，也可以改为except:
print('other errors')		
~                           

# 当没有输入任何东西按回车导致不合法时
Place input the first num: 
other errors

# 当除数为0时
Place input the first num: 6
Place input the second num: 0
The second number connot be zero.

# 正常操作
Place input the first num: 6
Place input the second num: 2
x/y= 3.0
3.0
```
#### 使用元组表示多个异常
可以用元组的形式将异常类型写在except后面。
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()
    print(c)
except (ZeroDivisionError,ValueError):	# 添加了输入校验
    print('The input must be a number and the second number connot be zero.')	
except Exception:
    print('other errors')
~                              


# 多次执行输出结果
Place input the first num: 6 
Place input the second num: 5
x/y= 1.2
1.2

Place input the first num: abc  
The input must be a number and the second number connot be zero.

Place input the first num: 6
Place input the second num: 0
The input must be a number and the second number connot be zero.

Place input the first num: 6/3
The input must be a number and the second number connot be zero.
```

### 抑制异常导致返回值无法外传
出错的时候，try/except的写法，函数的返回值不会继续往外传播，所以如果使用try/except这样的抑制异常功能，此时程序内部不应该依赖返回值。
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()+1
    print(c)
except ZeroDivisionError:
    print('The second number connot be zero.')
                                  

# 第一次执行：函数的返回值有继续向外传播
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
3.0		# 函数执行后的返回值+1

# 第二次执行：出错后函数的返回值没有继续向外传播，所以返回值+1的语句被
Place input the first num: 6
Place input the second num: 0
The second number connot be zero.
```

## 异常上下文
有时，程序其实不止有一个异常发生，一个异常可能是由前面的多个异常导致的，此时我们可以用raise来同时抛出多个异常信息。
```
# 分步引起异常：将字符串和数字相加，引起了TypeError和ValueError
>>> x='abc'
>>> x+1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate str (not "int") to str
>>> int(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: 'abc'
```
例如int(x)+1的例子，当x指向的是字符串时，此处有两个异常。字符串abc不能被int转换成数字的ValueError: invalid literal for int() with base 10: 'abc'，然后再是字符串不能和数字相加的TypeError: can only concatenate str (not "int") to str，所以此处其实是有两个异常，但程序内执行通常只会抛出一个异常信息，此时可以用raise抛出多个异常信息。
```
# 抛出多个异常信息
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()+1
    print(c)
except (ZeroDivisionError,ValueError):
    print('The input must be a number and the second number cannot be zero.')
    raise TypeError
except Exception:
    print('other errors')

print('Hello world!')
~                            

# 输出结果：同时引发了TypeError和ValueError
Place input the first num: 6
Place input the second num: abc
The input must be a number and the second number cannot be zero.

Traceback (most recent call last):
  File "python3_u8_error.py", line 16, in <module>
    c=div()+1
  File "python3_u8_error.py", line 10, in div
    y=int(input('Place input the second num: '))
ValueError: invalid literal for int() with base 10: 'abc'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "python3_u8_error.py", line 20, in <module>
    raise TypeError
TypeError
```

## 显示式捕获异常
当不想额外设置捕获异常后的异常信息显示，让使用者直接知道系统本身给的异常信息，同时还想程序继续执行下去不中断，可在except后面加as e来收集捕获到的错误，再用print显示其错误信息到屏幕上。</br>
一般来说，这种方法会比较好，因为前面的那些写法会让人不知道到底是哪里出现错误，甚至隐藏了程序编写者意料之外的错误，所以还是except Exception as e让程序无论在发生什么错误的情况下都继续运行下去，但同时输出系统给的错误信息的这种做法比较靠谱。</br>
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()+1
    print(c)
except (ZeroDivisionError,ValueError) as e:
    print(e)
except Exception:
    print('other errors')

print('Hello world!')


# 正常的输出
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
3.0
Hello world!

# 输出了ZeroDivisionError的错误信息
Place input the first num: 6
Place input the second num: 0
division by zero
Hello world!

# 输出了ValueError的错误信息
Place input the first num: 
invalid literal for int() with base 10: ''
Hello world!
```


## 无异常处理 try/except/else
### try/except/else
在try/except后面添加else子句，可以让程序执行后，接着执行else后面的代码。
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

try:
    c=div()+1
    print(c)
except (ZeroDivisionError,ValueError) as e:
    print(e)
except Exception:
    print('other errors')
else:			# try后面的语句正常执行后，执行此代码块
    print('sucessful')

print('Hello world!')

# 输出结果
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
3.0
sucessful		# 执行成功提示
Hello world!
```

### try/except/else结合while True/break
try/except/else结合while True/break使用可阻挡所有不符合预想的情况，直到用户的行为符合设计的预想，否则一直重复执行代码。</br>
* 使用except: </br>
这种做法可以捕获所有料想到或没料想到的错误，但不会有错误提示。</br>
* 使用except Exception as e：</br>
可以捕捉由Exception派生出来的异常，有错误提示，但不能捕捉由旧式字符串引发的异常和不由Exception派生出来的异常。</br>
```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

while True:
    try:
        c=div()+1
        print(c)
    except (ZeroDivisionError,ValueError) as e:
        print(e)
    except:
        print('some unexpected errors')
    else:
        break

print('Hello world!')

# 正常执行
Place input the first num: 6
Place input the second num: 2
x/y= 3.0
4.0
Hello world!

# 各种不合法输入执行结果
lace input the first num: 3   
Place input the second num: 0
division by zero
Place input the first num: 3
Place input the second num: 
invalid literal for int() with base 10: ''
Place input the first num: 
invalid literal for int() with base 10: ''
Place input the first num: asf
invalid literal for int() with base 10: 'asf'
Place input the first num: 6
Place input the second num: 3
x/y= 2.0
3.0
Hello world!
```

## 无条件执行 try/finally
finally和try配套，无论try后面的子句是否出错，都会执行finally后面的语句。</br>
finally通常用于变量的清理或文件、网络套接字等的关闭。</br>

```
def div():
    x=int(input('Place input the first num: '))
    y=int(input('Place input the second num: '))
    x/y
    print('x/y=',x/y)
    return x/y

i=0
while True:
    try:
        c=div()+1
        print(c)
    except (ZeroDivisionError,ValueError) as e:
        print(e)
    except:
        print('some unexpected errors')
    else:
        break
    finally:
        i=i+1
        print('This is the {} times to execute this function.'.format(i))

print('Hello world!')

# 输出结果：finally计算循环执行次数
Place input the first num: 
invalid literal for int() with base 10: ''
This is the 1 times to execute this function.
Place input the first num: 
invalid literal for int() with base 10: ''
This is the 2 times to execute this function.
Place input the first num: 
invalid literal for int() with base 10: ''
This is the 3 times to execute this function.
Place input the first num: 2
Place input the second num: 
invalid literal for int() with base 10: ''
This is the 4 times to execute this function.
Place input the first num: 36
Place input the second num: 6
x/y= 6.0
7.0
This is the 5 times to execute this function.
Hello world!
```

## try/except用于检验和调用对象
假设对象存在，然后直接调用。如果对象不存在则抛出异常提醒，这样做比较省事。</br>
```
# 查找字典中的键是否存在并输出
>>> dictStudentList
{'Sharon': 26, 'Tom': 24, 'Ben': 24}
>>> def findStudent(name):
...     try:
...         print('The student age is {}'.format(dictStudentList[name]))
...     except Exception as e:
...         print('error:',e)
... 
>>> findStudent('Tom')
The student age is 24
>>> findStudent('lulu')
error: 'lulu'
```
```
# 查找对象是否包含某属性
class Student:
    def __init__(self,name): 
        self.name=name
    def sayHi(self):
        print('Hello!',self.name)  
    
student1=Student('Tom') 
student2=Student('Ben') 

try:
   student1.name
except Exception as e:
   print(e)
else:
   student2.sayHi()

try:
    student2.age
except Exception as e:
    print(e)
else:
    print('The attribute of this student is exist.')

# 输出结果
Hello! Ben
'Student' object has no attribute 'age'
```



## 发出及过滤警告
### warn() 发出警告
模块warning，函数warn()可发出警告。</br>
参数：字符串，类名</br>
返回：类名（默认为__main__）和警告信息</br>
```
>> import warnings
>>> warnings.warn('Somethong Wrong!')
__main__:1: UserWarning: Somethong Wrong!
```

### filterwarnings() 过滤警告
模块warning，函数filterwarnings()可过滤警告。
