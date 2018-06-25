# Python 函数
函数是结构化编程的核心。</br>
函数是组织好的，可重复使用的，用来实现单一，或相关联功能的代码段。</br>
函数能提高应用的模块性，和代码的重复利用率。</br>

## 函数调用查询
### 内置函数 callable(obj)
callable(obj) ：判断某个对象是否可调用</br>
参数：对象</br>
返回值：布尔型</br>
```
>>> x=1
>>> import math
>>> y=math.sqrt
>>> callable(x)    # x不能被调用
False
>>> callable(y)    # y可以被调用
True
>>> y(4)         # 调用y
2.0
```
## 自定义函数
### 自定义函数格式
* 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号 ()。</br>
* 任何传入参数和自变量必须放在圆括号内，圆括号之间可以用于定义参数。</br>
* 圆括号后接冒号：表示函数代码块开始，代码块以4空格缩进。</br>
* 代码块的第一行可使用文档字符串docstring来进行函数说明，由单引号括起，不需要#号。</br>
* return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。</br>
```
def fibs():
    'Fibonacci sequence'
    num=int(input('How many Fibonacci number do you want? '))
    Fib_seq=[0,1]
    for n in range(num-2):
        Fib_seq.append(Fib_seq[-2]+Fib_seq[-1])
    return print(Fib_seq)

fibs()    # 调用函数fibs


How many Fibonacci number do you want? 10
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```
交互式窗口执行，以“不再缩进+回车”结束代码块。
```
>>> def fibs(num):
...     'Fibonacci sequence'
...     Fib_seq=[0,1]
...     for n in range(num-2):
...         Fib_seq.append(Fib_seq[-2]+Fib_seq[-1])
...     return Fib_seq
... 
>>> fibs(16)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610]
```
### print(vars)和return vars的区别
#### return 结束函数
return结束函数的执行而print则不会。
```
>>> def say_hi():
...     print('hello')
...     return
...     print('world')
... 
>>> say_hi
<function say_hi at 0xb70846ec>
>>> say_hi()
hello
>>> 
```
#### 打印到屏幕和函数返回值上的区别
* 交互式执行</br>
交互式执行会把函数的返回值直接打印再屏幕上，而不需要写额外的print。</br>
print(vars) 仅仅是把vars的值打印在屏幕上，整个函数的返回值是None；</br>
return vars屏幕显示的值是函数的返回值，返回值可以被变量接走继续使用。</br>
```
# 交互模式下执行print(vars)
>>> def test_print(x,y):
...     print(x+y)
...     return
... 
>>> b=test_print(3,4)
7
>>> b+1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'
>>> 
>>> type(b)
<class 'NoneType'>
```
```
# 交互模式下执行 return vars
>>> def test_return(x,y):
...     return x+y
... 
>>> c=test_return(3,4)
>>> c
7
>>> c+1
8
>>> type(c)
<class 'int'>
```
```
# 交互模式下执行函数
>>> test_print(3,4)
7
>>> test_return(3,4)
7
```

* 脚本模式下执行函数</br>
脚本模式下，只有print语句可以使变量的值显示在屏幕上，而return则不能。</br>
```
# 脚本模式下print(vars)
def test_print(x,y):
    print(x+y)
    return

test_print(3,4)				# 执行test_print()，由于有print(x+y)，所以会把x+y的值打印再屏幕上
print(test_print(3,4))		# 执行test_print()，将test_print()的返回值打印到屏幕上

# 脚本模式下 return vars
def test_return(a,b):
    return a+b

test_return(3,4)			# 仅是执行了test_return()，结果不会输出到屏幕上 
print(test_return(3,4))		# 把test_return()的返回值输出到屏幕
```

```
# 执行脚本的屏幕显示结果
7						# 执行test_print()后，此值为print(x+y)的输出值
7						#  print(test_print(3,4))先执行test_print()，此值为print(x+y)的输出值
None				# 此值为函数test_print()的返回值的输出值
7						# 此值为函数test_return()的返回值的输出值
```



## 参数传递
Python变量本身没有类型，而变量指向的对象有类型。</br>
参数的对象类型可分为两大类：不可变类型immutable、可变类型mutable。</br>
### 不可变类型immutable
* 不可变类型对象种类：string 字符串、tuple 元组、number 数字</br>
* 对象的变量名：</br>
变量名只是指向对象，变量名对应值的更改只是指向的对象改变了，而不是对象本身改变了。</br>
a=5 把变量名a指向了5，然后执行a=10，a重新指向10，对象数字5和10本身都不会被改变。</br>
a=5和b=5，这种情况是变量a和b均指向了数字5，a和b是相等的。</br>
```
>>> a=5
>>> a=10
>>> a
10
```
```
>>> a=5
>>> b=5
>>> a == b
True
```
```
>>> b=7
>>> a=8  # 将a指向8
>>> a=b  # 将a指向b的对象7
>>> b
7
>>> a
7
```
* Python函数的immutable类型参数传递：</br>
函数外部的immutable类型参数传递给函数后，在函数内部参数变量名的指向可以改变，但函数结束后参数变量名的指向会恢复到进入函数之前的样子，而不受函数影响。</br>
```
>>> def num(n):
...    c=5
...    return c
... 
>>> c=9
>>> num(c)
5
>>> c
9
```

### 可变类型 mutable
* 可变类型对象种类：list 列表、dictionary 字典</br>
* 对象的变量名称：</br>
list1=[1,2,3] 把变量名list1指向了列表[1,2,3]，n=list1把变量名n指向了list1，n也同时指向了列表[1,2,3]；</br>
变量list1和n指向的是同一个列表[1,2,3]，所以list1[index]和n[index]都会修改列表[1,2,3]的值；</br>
可变的是列表[1,2,3]的值，字典同理。</br>
list2=[1,2,4]和n2=[1,2,4]，这种情况是list2和n2分别指向了两个元素均为1,2,4的列表，这两个列表不是同一个列表，它们相等而不同。</br>
```
>>> list1=[1,2,3]
>>> n=list1
>>> n
[1, 2, 3]
>>> n[0]=2
>>> list1==n
True
>>> list1 is n
True
>>> n
[2, 2, 3]
>>> list1
[2, 2, 3]
>>> 
```
```
>>> list2=[1,2,4]
>>> n2=[1,2,4]
>>> n2==list2
True
>>> list2 is n2
False
>>> 
>>> n2[0]=2
>>> n2
[2, 2, 4]
>>> list2
[1, 2, 4]
```
```
>>> n3=[3,4,5]
>>> list3=[3,4,5]
>>> n3=list3
>>> n3 is list3
True
>>> n3[0]=2
>>> n3
[2, 4, 5]
>>> list3
[2, 4, 5]
>>> 
```

* Python函数的mutable类型参数传递：</br>
函数外部的mutable类型参数传递给函数后，函数结束后参数内部的一些值会受到函数影响。

```
>>> def changelist(ls):
...     ls[0]='hello'
...     return ls
... 
>>> list4=[1,2,3]
>>> list4
[1, 2, 3]
>>> changelist(list4)
['hello', 2, 3]
>>> list4
['hello', 2, 3]
>>> 
```

```
>>> def changelist2(ls):
...    n4=ls
...    n4[1]='world'
...    return ls
... 
>>> list4
['hello', 2, 3]
>>> n4
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'n4' is not defined
>>> changelist2(list4)
['hello', 'world', 3]
>>> list4
['hello', 'world', 3]
>>> n4
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'n4' is not defined
>>>
```

```
>>> def changelist3(ls):
...    n5=ls
...    n5[2]='!'
...    return ls,n5
... 
>>> list4
['hello', 'world', 3]
>>> n5
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'n5' is not defined
>>> changelist3(list4)
(['hello', 'world', '!'], ['hello', 'world', '!'])
>>> list4
['hello', 'world', '!']
>>> n5    # 注意：n5是在函数内定义的，尽管写了return，退出函数后n5依然是没有定义
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'n5' is not defined
>>> 
```



## 函数调用时的参数输入形式
调用函数时可使用以下几种输入形式：</br>
位置参数、关键字参数、默认参数、任意长度参数</br>
不应结合使用位置参数和关键字参数！！</br>

### 位置参数
位置参数：根据输入的值的先后顺序来给参数一一赋值的，函数调用时参数的输入顺序会影响函数输出结果。
```
>>> def person(name,age):
...     print('name:',name)
...     print('age:',age)
... 
>>> person('Sharon',25)  # 注意：函数调用时的输入顺序
name: Sharon
age: 25
>>> person(25,'Sharon')
name: 25
age: Sharon
>>> 
```


### 关键字参数
关键字参数：调用函数时，指定输入值和函数参数名的对应关系。这样调用可以不必关注参数的输入顺序。
```
>>> def person(name,age):
...     print('name:',name)
...     print('age:',age)
... 
>>> name1=input('Place input your name:')
Place input your name:Sharon
>>> age1=input('Place input your age:')
Place input your age:25
>>> person(age=age1,name=name1)  # 注意：没有按照变量出现的顺序输入
name: Sharon
age: 25
>>> 
```

### 默认参数
默认参数：调用时给参数设置默认值，这样调用参数时则可不提供参数。</br>
参数的默认值需要在定义函数时写在圆括号内。</br>
```
>>> def func(bear1='grizz',bear2='panda',bear3='icebear'):
...     print('{} is big brother and {} is second in line. {} is the smallest.'.format(bear1,bear2,bear3))
... 
>>> func()
grizz is big brother and panda is second in line. icebear is the smallest.
>>> func('Sharon','Jack','Ben')
Sharon is big brother and Jack is second in line. Ben is the smallest.
```


### 任意长度参数
任意长度参数：允许用户在调用函数时输入任意多个参数。在函数定义时，在参数前面带星号*或**，收集多个参数，参数的所有值会被放在一个元组或字典当中。</br>
#### 当带星号参数被置于最后
在定义函数时，把带星号参数被置于最后，收集余下所有位置的参数：</br>
* 定义函数时，被带星号参数放置在最后；</br>
* 调用函数时，采用位置参数的形式，带星号参数会收集余下位置的参数。</br>
当带星号参数被置于最后，被收集参数会被放置在元组中。</br>

```
>>> def group(groupname,*membernames):
...     print('Your group name:',groupname)
...     print('Your members\'name: ',membernames)  # 参数调用时不需要带星号
... 
>>> group('team one','Sharon','lulu','Ben')
Your group name: team one
Your members'name:  ('Sharon', 'lulu', 'Ben')  # 星号把余下的参数全部放置在元组中
>>> 
```

```
>>> group('team one','Sharon','lulu','Ben',201132000) 
Your group name: team one
Your members'name:  ('Sharon', 'lulu', 'Ben', 201132000)  # 星号收集了余下所有值，于是发生了错误
```

#### 当带星号的参数不被置于最后
在定义函数时，也可不把带星号的参数置于最后，把带星号参数后面的参数设置为关键字参数：</br>
* 定义函数时，把带星号参数放置在中间；</br>
* 调用函数时，把不想被带星号参数收集的输入值设置为关键字参数。</br>
带星号参数不会收集关键字参数，收集到的值会被放置在元组中。</br>

```
>>> def group2(groupname,*membernames,number):
...     print('Your group name:',groupname)
...     print('Your members\'name: ',membernames)
...     print('Your group number: ',number)
... 
>>> group2('team one','Sharon','lulu','Ben',201132000)  # 计算机无法判断导致出错
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: group2() missing 1 required keyword-only argument: 'number'
>>> group2('team one','Sharon','lulu','Ben',number=201132000)  # 设置为关键字参数
Your group name: team one
Your members'name:  ('Sharon', 'lulu', 'Ben')
Your group number:  201132000
>>> 
```

#### 让带星号参数收集关键字参数
想让带星号参数收集关键字参数，可将*号改为两个星号**以收集关键字参数：</br>
* 定义函数时，给需要收集多个输入值的参数带两个星号；</br>
* 定义函数时，带两个星号的参数必须被放置在最后；</br>
* 调用函数时，把想被带两个星号参数收集的输入值设置为关键字参数；</br>
* 调用函数时，要被想被收集的输入值放最后且要有不同的关键字（注意和普通的关键字参数不用）。</br>
带两个星号的参数会收集关键字参数，收集到的值会被放置在字典中，关键字会被作为字典的key。</br>

```
# 带两个星号的参数后面不能再有参数，因为逻辑上和不收集关键字参数的带*参数的设计冲突了。
>>> def group3(groupname,**membernames,number):
  File "<stdin>", line 1
    def group3(groupname,**membernames,number):
                                            ^
```

```
>>> def group3(groupname,number,**membernames):
...     print('Your group name:',groupname)
...     print('Your members\'name: ',membernames)
...     print('Your group number: ',number)
... 
>>> group3('team one',201132000,member1='Sharon',member2='lulu',member3='Ben') 
# 被收集的输入参数的名字必须不一样
Your group name: team one
Your members'name:  {'member1': 'Sharon', 'member2': 'lulu', 'member3': 'Ben'}  # 被收集到了字典当中
Your group number:  201132000
>>> 
```

```
>>> group3('team one',number=201132000,member1='Sharon',member2='lulu',member3='Ben')
# 注意两种关键字参数的区别，number用的就是参数名，而groupmembers则不是
Your group name: team one
Your members'name:  {'member1': 'Sharon', 'member2': 'lulu', 'member3': 'Ben'}
Your group number:  201132000
>>> 
```



## 参数分配：解包参数序列
有时想把序列中的值作为不同的参数传递给函数作输入，此时需要先给序列解包：</br>
* 在函数调用时进行操作，使用星号*解包列表，使用两个星号**解包字典；</br>
* 被解包的序列必须位于函数调用时，所有参数的末尾；</br>
* 不能在函数定义和函数调用时同时使用号或**，这样会导致只传递元组和字典。</br>

```
>>> def plus(x,y,z):
...     print('x+y+z=',x+y+z)
... 
>>> a=[3,4,5]
>>> plus(*a)  # 解包列表
x+y+z= 12
>>> 
```

```
>>> def studentID(classnum,stuID1,stuID2):
...     print('class {}: student1 ID {}'.format(classnum,stuID1))
...     print('class {}: student2 ID {}'.format(classnum,stuID2))
... 
>>> stuID={'stuID1':26,'stuID2':96}  # 要解包的字典的key必须对应函数定义的参数名，被传递的是value
>>> studentID(2,**stuID)
class 2: student1 ID 26
class 2: student2 ID 96
>>> 
```

在不用**下改写一下上面的程序：

```
>>> def course(IDdict):
...     for classnum in IDdict:
...         student_dict=IDdict[classnum]
...         for name in student_dict:
...             print('class {}:{} ID {}'.format(classnum,name,student_dict[name]))
... 
>>> student_ID
{1: {'Lily': 2, 'Ben': 26}, 2: {'Sharon': 26, 'Lu': 30}, 5: {'Jack': 40}}
>>> course(student_ID)class 1:Lily ID 2
class 1:Ben ID 26
class 2:Sharon ID 26
class 2:Lu ID 30
class 5:Jack ID 40
```


## 作用域
命名空间：一个存储变量名及其值的字典。</br>
除全局作用域外，每个函数调用都会创建一个命名空间。</br>
### 查看命名空间
#### vars() 查看命名空间字典
vars()：内置函数，可查看程序的命名空间。返回的字典内容取决于vars()执行的位置。</br>
参数：无</br>
返回值：当前程序的命名空间字典</br>

```
# 查看所有全局变量的命名空间
>>> vars()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'course': <function course at 0xb70badf4>, 'student_ID': {1: {'Lily': 2, 'Ben': 26}, 2: {'Sharon': 26, 'Lu': 30}, 5: {'Jack': 40}}, 'x': 1, 'y': 2}
>>> 
```

```
# 查看局部变量的命名空间
>>> def vars_func2():
...     x=2
...     y=8
...     print(vars())
... 
>>> vars_func2()
{'y': 8, 'x': 2}
>>> 
```

查看单个变量对应的值</br>
需要先把vars()输出的字典赋值给一个变量，再使用该字典进行单个变量对应值的查询，不能直接使用vars()查。</br>

```
# 错误示范
>>> vars('x')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: vars() argument must have __dict__ attribute
```
```
# 正确示范
>>> vars_dict=vars()
>>> vars_dict['x']
1
>>> 
```

#### 全局变量与局部变量
全局变量与局部变量可以同名，但在子函数里，局部变量的值会覆盖全局变量的值，不建议这么做。</br>
##### 函数中查看全局变量和局部变量

```
>>> def vars_func():
...     x=3
...     y=7
...     print(locals())					# 查看局部变量
...     print(globals()['x'],globals()['y'])    # 查看全局变量
... 
>>> vars_func()
{'y': 7, 'x': 3}
1 2
>>> 
```

##### 函数中给全局变量或函数外变量赋值
* 语句 global</br>
global variable: </br>
可在函数中给全局变量赋值，但是要先声明再赋值且全局变量已存在。</br>

```
>>> def vars_func3():
...     global z    # 需先声明再赋值
...     z=9
... 
>>> z=3			 # 全局变量z必须已存在
>>> z
3
>>> vars_func3()
>>> z
9
>>> 
```

* 语句 nonlocal</br>
nonlocal variable：</br>
可在函数内部给函数外部的变量赋值（非全局变量），常用在函数嵌套的情况中，Python3新增。</br>

```
>>> def vars_func4():
...     testing=3				# 外部变量testing必须已存在
...     print(testing)
...     def vars_func5():
...         nonlocal testing     # 需先声明后赋值
...         testing=8
...         print(testing)
...     vars_func5()
...     print(testing)
... 
>>> vars_func4()
3
8
8
>>> testing					# 不影响全局变量
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'testing' is not defined
>>> 
```

## 递归
递归：函数自己引用自己，此处用的是return。</br>

```
>>> def test_space():
...     name=input('Place input your name:')
...     if name=="" or name.isspace():
...         return test_space()
...     else:
...         print('hello,{}!'.format(name))
...         return
... 
>>> test_space()
Place input your name:
Place input your name:
Place input your name:
Place input your name:Sharon
hello,Sharon!
>>> 
```
