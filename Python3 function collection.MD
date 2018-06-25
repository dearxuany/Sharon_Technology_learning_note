# Python 常用函数积累
## type(variable) 类型
查看变量类型
```
>>> type(1)
<class 'int'>
>>> num=input('number: ')
number: 7
>>> type(num)
<class 'str'>
```
## range(num1,num2,step) 范围
建立以num1为开头、num2为结尾但不包括num2的一个范围，可用步长step跳过一些数。起始值包含在范围内，结束值不包含。</br>
常和for循环合用，迭代时，要倒序的数字范围必须加步长，且步长为负。</br>
参数：起始值，结束值，步长</br>
返回：一个范围</br>
```
>>> range(1,6)
range(1, 6)
>>> list(range(1,7))
[1, 2, 3, 4, 5, 6]
>>> list(range(1,7,2))
[1, 3, 5]
>>> 
>>> list(range(99,93))
[]
>>> list(range(99,93,-1))
[99, 98, 97, 96, 95, 94]
```


```
list1=list(range(0,10))    #建立一个由0到10组成的列表
for n in list1:
    print(n)

0
1
2
3
4
5
6
7
8
9
```
```
i=input('Place input a number: ')
c=0
# 计算把由i到99的所有整数相加。
for i in range(1,100):
    c=c+i
print(c)

Place input a number: 1
4950
```
```
a=0
b=1
# 斐波纳契数列，输出20位
for y in range(20):
    print(b)
    c=a
    a=b
    b=c+b


1
1
2
3
5
8
13
21
34
55
89
144
233
377
610
987
1597
2584
4181
6765
```
## zip(seq1，seq2) 链接
zip(seq1，seq2)：会将seq1和seq2的元素值按照index顺序一对一连起来，并返回一个元组。</br>
参数：seq1,seq2,seq3,...,seqn</br>
返回值：多个连接seq1，seq2数据的元组，格式为（value1,value2,...,valuen），是适合迭代的对象，要用list()将它们组成列表。</br>
注意：zip(seq1,seq2,seq3,...,seqn) 适用于所有类型序列，可同时连接无数多个序列，目标序列长度不一致时，连接所得元组长度由长度最短的目标序列决定。</br>
```
>>> list2=['Sharon','Jack','Ben','lulu']
>>> list3=[19930226,19960102,19960102,20150905]
>>> zip(list2,list3)
<zip object at 0xb711ba6c>
>>> list(zip(list2,list3))
[('Sharon', 19930226), ('Jack', 19960102), ('Ben', 19960102), ('lulu', 20150905)]
```

```
list2=['Sharon','Jack','Ben','lulu']
list3=[19930226,19960102,19960102,20150905]
list4=[25,22,22,3]

# 由于zip()返回值的性质，必须根据元素值来进行迭代。
for name,birthday,age in zip(list2,list3,list4):
    print("{} who  was born on {} is {} yearsold this year.".format(name,birthday,age))

Sharon who  was born on 19930226 is 25 yearsold this year.
Jack who  was born on 19960102 is 22 yearsold this year.
Ben who  was born on 19960102 is 22 yearsold this year.
lulu who  was born on 20150905 is 3 yearsold this year.
```

## enumerate(seq) 枚举
enumerate(seq) : 根据一个序列seq，创建多个以（index,value）为格式的元组，用于序列的迭代。</br>
参数：seq</br>
返回值：多个以（index,value）为格式的元组</br>
```
>>> list1=list('hello')
>>> enumerate(list1)
<enumerate object at 0xb70b3c34>
>>> list(enumerate(list1))
[(0, 'h'), (1, 'e'), (2, 'l'), (3, 'l'), (4, 'o')]
>>> 
```
```
list5=[('Sharon', 19930226, 25), ('Jack', 19960102, 22), ('Ben', 19960102, 22), ('lulu', 20150905, 3)]
# 根据索引和元素值迭代
for index,name_birth_age in enumerate(list5):
    print('number {}: {} '.format(index,name_birth_age))

number 0: ('Sharon', 19930226, 25) 
number 1: ('Jack', 19960102, 22) 
number 2: ('Ben', 19960102, 22) 
number 3: ('lulu', 20150905, 3) 
```
## sorted(seq) 排序
sorted(seq) ：内置函数，序列元素排序，适用于任何序列或可迭代对象，不修改原序列，返回排序后的新序列。</br>
参数：序列</br>
返回值：序列排序后生成的列表</br>
注意：列表方法 lst.sort(cmp,key,reverse) 会修改原序列，而内置函数sorted(seq)则不会。</br>
```
>>> seq1="hello world"
>>> sorted(seq1)
[' ', 'd', 'e', 'h', 'l', 'l', 'l', 'o', 'o', 'r', 'w']
>>> seq1
'hello world'
>>> list1=list(seq1)
>>> list1
['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']
>>> sorted(seq1)
[' ', 'd', 'e', 'h', 'l', 'l', 'l', 'o', 'o', 'r', 'w']
>>> list1
['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']
>>> list1.sort()
>>> list1
[' ', 'd', 'e', 'h', 'l', 'l', 'l', 'o', 'o', 'r', 'w']
```
## reversed(seq) 反向
reversed(seq)：内置函数，反向排序，适合于任何序列或可迭代对象，不影响原序列，返回的不是列表而是一种可迭代对象。</br>
参数：sequence</br>
返回值：反向排序后的可迭代对象</br>
注意：和列表方法lst.reverse()不同，reversed(seq)可用于所以序列且不修改原序列。</br>
```
>>> seq1
'hello world'
>>> reversed(seq1)
<reversed object at 0xb704c8ec>
>>> ''.join(reversed(seq1))
'dlrow olleh'
```
## exec(str) 执行
exec(str)：内置函数，以前是语句，可将用户提供的字符串当做代码执行。</br>
参数：字符串</br>
返回值：无（取决于当做代码执行的字符串是否有返回值）</br>
注意：常用于在线编译器，这样做很危险，不建议使用。</br>
```
>>> exec("print('hello')")
hello
# exec()本身不返回任何东西，只是执行了print，返回值是print的返回值
```
```
str_input=input("Place input a command to execute: ")
exec(str_input)

Place input a command to execute: print('hello')
hello
```
## eval 
eval(str) ：内置函数，计算由字符串表示的Python表达式，有返回值。</br>
参数：字符串</br>
返回值：计算结果</br>
注意：eval(str)是有返回值的。eval(str)也是不安全的，不建议使用。</br>
```
>>> exec('1+1')
>>> eval('1+1')
2
>>> 
```
```
str_input=input("Place input an arithmetic expression: ")
print(str_input)
print(eval(str_input))

Place input an arithmetic expression: 9*3-4
9*3-4
23
```

## vars() 查看命名空间字典
vars()：内置函数，可查看程序的命名空间。返回的字典内容取决于vars()执行的位置。</br>
参数：无</br>
返回值：当前程序的命名空间字典</br>

```
# 查看所有全局变量的命名空间
>>> vars()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'course': <function course at 0xb70badf4>, 'student_ID': {1: {'Lily': 2, 'Ben': 26}, 2: {'Sharon': 26, 'Lu': 30}, 5: {'Jack': 40}}, 'x': 1, 'y': 2}
>>> 

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

## 函数中查看全局变量 globals() 和局部变量 locals()
 globals()返回的是全局变量字典，locals()返回的是当前函数局部变量字典。
 
```
>>> def vars_func():
...     x=3
...     y=7
...     print(locals())					        # 查看局部变量
...     print(globals()['x'],globals()['y'])    # 查看全局变量
... 
>>> vars_func()
{'y': 7, 'x': 3}
1 2
>>> 
```
