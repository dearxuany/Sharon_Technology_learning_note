# Python 语句：赋值、条件、循环及其他语句
## 缩进
Python强制要求代码缩进来嵌套逻辑（据说是为了可读性以及培养编程者的良好习惯）。</br>
同一个代码块中，各行的代码缩进量必须相同。</br>
* 空格：标准也更好，使用4个空格。
* 制表符：一个制表符，相当于8个空格。
* ：冒号：冒号可用于使一个代码块里的所有代码行均有相同缩进。</br>
</br>
根据PEP的规定，必须使用4个空格来表示每级缩进。</br>
（不清楚为何要规定是4个空格，在实际编写中可以自定义空格数，但是要满足每级缩进间空格数相等。）</br>
使用Tab字符和其它数目的空格虽然都可以编译通过，但不符合编码规范。</br>
支持Tab字符和其它数目的空格仅仅是为兼容很旧的的Python程序和某些有问题的编辑程序。</br>
空格和制表符都可以用于缩进，但是程序里不能两种混用，否则会造成判断错误。编辑器不同也会导致错误发生。</br>

## 赋值
### 序列解包（可迭代对象解包）
序列解包：可利用序列同时给多个变量赋值。</br>

```
import math   
a=math.sqrt(100)
b=math.sqrt(10)  
c=math.sqrt(4)

results=[a,b,c]
print("calculation results=",results)
x,y,z=results
print('x=',x,'y=',y,'z=',z)

calculation results= [10.0, 3.1622776601683795, 2.0]
x= 10.0 y= 3.1622776601683795 z= 2.0
```
注意：</br>
要解包的序列的元素个数必须与等号的左边的变量个数相同（否则会出错），可用星号*打包多余的值。</br>
等号右边可以是任何类型的序列，但带星号的变量的值一定为一个列表，等号两边个数相同时也是如此。</br>

```
str1='How are you ?'
first,*second,third=str1.split()		# 用*星号收集多余项
print(first)
print(second)
print(third)
first,second,*third=str1.split()		# 注意*星号的位置
print(first)
print(second)
print(third)

How
['are', 'you']					# 类型为列表
?
How
are
['you', '?']
```

### 链式赋值
```
>>> o=p=3	# 链式赋值
>>> o
3
>>> p
3
>>> p=4
>>> o=p
>>> o
4
>>> p
4
```

### 增强赋值
#### 算术运算
双目运算符可以使用增强赋值的写法，可使代码更简洁紧凑，可读性高。
```
>>> p
4
>>> p=p+1
>>> p
5
>>> p+=1
>>> p
6
>>> p+=2
>>> p
8
>>> p*=2
>>> p
16
>>> p
16
```

#### 其他类型
其他数据类型也可使用增强赋值。
```
>>> w='hello'
>>> w+=' world'
>>> w
'hello world'
>>> w*2
'hello worldhello world'
>>> w
'hello world'
>>> w*=2
>>> w
'hello worldhello world'
```

## 条件语句
详情 https://docs.python.org/3.7/reference/compound_stmts.html#while</br>
### 布尔型
假 False  0 ：标准值False和None，各数据类型的值0，空序列() [] “”、空字典{}，会被Python解释为假。</br>
真 True	1 ：除假以外的其他值。</br>
```
>>> True	# 注意大小写
True
>>> true
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'true' is not defined
>>> TRUE
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'TRUE' is not defined
>>> True+False+2	# 布尔型可以计算
3
```

bool() 转换为布尔型</br>
Python中任何值都可以被转换为布尔型，bool()可以实现转换，但不需要专门去这么做，Python会自动转换。</br>
```
>>> bool(77)
True
>>> bool({})
False
>>> bool(0)
False
```

### 条件判断语句
#### 条件判断语句 if... elif... else...
if 条件判断1，elif 条件判断2，else 其他；</br>
注意大小写，语句全为小写字母；</br>
每个条件以冒号：结束表示满足条件后要执行的代码块的开始；</br>
条件判断注意中间的空格，冒号前没有空格，格式为‘obj 运算符 obj:’；</br>
用缩进来划分代码块，Python要求是4个空格；</br>
Python中，没有switch -case语句。</br>
```
num=float(input('Place input a number: '))

if num > 0:
    print('The number is bigger than 0.')
elif num < 0:
    print('The number is smaller than 0.')
else:
    print('The number is 0.')


Place input a number: 1
The number is bigger than 0.

Place input a number: 0.1
The number is bigger than 0.

Place input a number: 0
The number is 0.

Place input a number: -1
The number is smaller than 0.
```
条件判断代码块嵌套：</br>
注意缩进！！4个空格！！不要空格和制表符混用！！</br>
```
num2=float(input('Place input a number: '))

if num2 > 0:
    print('The number is bigger than 0.')
    if num2 < 10:
        print('The number is within 1 to 10.')
    elif num2 > 10:
        print('The numer is bigger than 10.')
    else:
        print('The number is 10.')
elif num2 < 0:
    print('The number is smaller than 0.')
else:
print('The number is 0.')


Place input a number: 3
The number is bigger than 0.
The number is within 1 to 10.

Place input a number: 11
The number is bigger than 0.
The numer is bigger than 10.

Place input a number: 10
The number is bigger than 0.
The number is 10.
```
#### 条件判断关键字 assert
关键字 assert：后接条件，判断某输入是否符合要求的条件。</br>
符合条件无输出，程序继续执行；当不符合条件时，可以让程序直接崩溃退出，不继续执行并输出错误提示。</br>
```
num2=float(input('Place input a number: '))
assert 0 <= num2 <= 10,'The number is not between 0 to 10.'
print('0 <= ',num2,'<= 10')

Place input a number: 99
 Traceback (most recent call last):
  File "python3_u5_statements.py", line 59, in <module>
    assert 0 <= num2 <= 10,'The number is not between 0 to 10.'
AssertionError: The number is not between 0 to 10.

Place input a number: 3
0 <=  3.0 <= 10
```

### 判断条件
#### 比较运算符
Python3 仅支持相同或相似类型对象的比较，如数字和字符串不能作比较。
```
x == y
x < y
x > y
x <= y
x >= y
x != y
x is y     # 判断x是否和y是同一对象
x is not y
x in y     # 判断x是否是容器y的成员
x not in y
```
* is 相同运算符</br>
== 用来检验两个对象是否相等，is用来检验两个对象是否相同。</br>
is 不能用于数、字符串、元组等不可变的基本值，Python内部对这些对象的处理方式会导致结果不可预测。</br>
```
>>> x=y=[1,3,4]
>>> z=[1,3,4]
>>> x == y
True
>>> x == z    # x和z的值相等
True
>>> x is y    # x和y指向同一列表
True
>>> x is z    # x和z指向的不是同一列表
False
```
以下为错误示范，不要这么做：
```
>>> 1 is 4
False
>>> 1 is 1
True
>>> 1 is 7
False
>>> a is 97
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
```
* in 成员资格运算</br>
```
>>> x=[2,3,4]
>>> y=[3,4]
>>> y in x
False
>>> 3 in x
True
>>> 1 in x
False
```
* 字符串的比较</br>
Python3中都是Unicode字符，字符按码点排序。</br>
ord(char)：可以获取char的Unicode码点。</br>
chr(num)：可将码点num转换成Unicode中对应的字符。</br>
字符串比较：比价Unicode码点大小，由左向右逐个比较。如果前面的已经可以作出判断，则不再比较后面的字符。</br>
```
>>> ord(%)
  File "<stdin>", line 1
    ord(%)
        ^
SyntaxError: invalid syntax
>>> ord('%')
37
>>> ord('9')
57
>>> ord(9)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: ord() expected string of length 1, but int found
>>> ord('x')
120
>>> ord('A')
65

>>> chr(97)
'a'
>>> chr(36)
'$'

>>> 'a'<'x'
True
>>> 'abc'<'ab'
False
>>> 'abc'<'abx'
True
>> 'a'<'#'
False
>>> 'a'<'A'
False
```
不同类型不能作比较，相同类型的可以：</br>
如果字符串和值为数字的字符串作比较，‘a’和‘97’比较，‘a’的码点97和‘9’的码点57比较，判断‘a’大于‘97’。由左到右逐个比较，直到可以作出判断为止。</br>
```
>>> 'a'== 97
False
>>> 'a'== '97'
False
>>> 'a'< '97'
False
>>> 'a'< 97
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'str' and 'int'
>>> 'a' > '97'
True
>>> 
```
* 其他序列比较</br>
相比较的序列的长度要相等。
```
>>> x=[2,3,4]
>>> y=[4,5]
>>> x > y
False
>>> x < y
True
>>> z=[2,[3,4]]
>>> x < z
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'int' and 'list'
>>> c=[3,[4,5]]
>>> z < c
True
>>> 
```
#### 布尔运算符（逻辑运算符）
布尔运算符：and、or、not</br>
同时符合两个条件，判断[1,10]的范围，可将if后面的条件写成 num >= 1 and num <= 10。</br>
Python 貌似不支持&&、||、！的写法。</br>

## 循环
### 循环语句 while
while 后面接条件，当符合条件时运行代码块；</br>
条件后面紧接冒号：表示代码块开始；</br>
4个空格缩进，以区分代码逻辑。</br>
```
# 斐波纳契数列
a=0
b=1
while b < 20:
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
```
```
# 检验变量是否为空

name=''  # 空为假，not name为真
while not name:
    name=input('Place input your name: ')
print('Hello, {}!'.format(name))

# 不输入任何东西，按回车
Place input your name: 
Place input your name: 
Place input your name: Sharon
Hello, Sharon!
# 输入一个空格，按回车
Place input your name:  
Hello,  !

# 将判断条件修改为 while not name or name.isspace():
# 输入一个空格，按回车
Place input your name:  
Place input your name:  
Place input your name: 
```

### 循环语句 for
for variable in sequence</br>
当variable的值和sequence中某元素的值时进入循环，遍历序列从当前值到序列结束的每一个元素，variable不设初始值默认为0，字典默认为遍历全部项。</br>
条件后面紧接冒号：表示代码块开始；</br>
4个空格缩进，以区分代码逻辑；</br>
for循环主要用于迭代（遍历），为序列中每个元素执行代码，或是控制循环次数；</br>
比while紧凑，可以用for不用while，for中的in是必须的；</br>
while比较灵活，如果想写如“n > 10”这样的判断条件，用while，for不能这样写。</br>
</br>
以下是错误示范</br>
不支持算术运算符，for必须用in。</br>
```
list1=list(range(0,10))    #建立一个由0到10组成的列表
n=0
for n < 10:
    print(list1[n])
    n=n+1

File "python3_u5_statements.py", line 78
    for n < 10:
          ^
SyntaxError: invalid syntax
```
Python中的for循环会遍历符合条件之后，序列当前值及其后面的所有元素，不能指定特定的数来进行循环。
```
a=0
b=1
for b in range(20):
    print(b)
    c=a
    a=b
b=c+b
# 结果会显示1至20而非斐波纳契数列，b变化的步长不是确定的，所以不能根据b的变化来进行循环。
```

以下为正确示范
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

### 跳出循环
#### break
```
a=0
list6=list(range(1,101))
# 由1加到100，当总和大于2500时停止相加，并输出当前和及当前i达到的数字。
for i in list6:
    a=a+i
    if a >= 2500:
        print('a=',a,'i=',i)    # 注意缩进
        break    # break写在if的代码块里面

a= 2556 i= 71
```

#### continue
跳过循环中的某些部分，继续后面循环。
```
j=0
h=0
list6=list(range(1,101))
# 把1到100中可以被3整除的数相加得h，把不能被3整除的数相加得j
for x in list6:
    if x % 3 == 0:
        h=h+x
        continue
    j=j+x
print(h,j)

1683 3367
```
#### while True/break
将循环的条件，写在while代码块里面，利用if跳出循环
```
j=0
h=0
x=0
# 把1到100中可以被3整除的数相加得h，把不能被3整除的数相加得j
while True:
    x=x+1
    if x % 3 == 0 and x <= 100:
        h=h+x
    elif x % 3 != 0 and x <= 100:
        j=j+x
    else:
        break
print(x,h,j)


101 1683 3367
```

#### else
else子句后接退出循环后的措施，while和for均能使用。
```
list2=['Sharon','Jack','Ben','lulu']
your_name=""
while your_name == "" or your_name.isspace():
    your_name=input('place input your name:')
else:
    print('Hello,{}!'.format(your_name))

place input your name:
place input your name:
place input your name:123
Hello,123!
```

## pass/del/exec 语句
### pass 略过
pass 语句：没有任何意思，就是什么都不做略过去，常用于未完成函数的占位。
```
say_hi=['morning','afternoon','night']
import time
print(time.strftime("%H:%M:%S")) #输出当前时间24小时格式
t=int(time.strftime("%H"))
if t >= 0 and t < 6:
    pass
elif t >= 6 and t <= 12:
    print("Good ",say_hi[0])
elif t > 12 and t <= 18:
    pass
else:
    pass

10:49:54
Good  morning
```
### del 删除
```
>>> x='hello'
>>> x
'hello'
>>> del x
>>> x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'x' is not defined
```

注意c的值，b变为None后不影响c，但是b的元素值改变会影响c。
```
>>> b
[1, 2, 3]
>>> c=b
>>> b[1]=4
>>> b
[1, 4, 3]
>>> c
[1, 4, 3]
>>> b=None
>>> b
>>> c
[1, 4, 3]
>>> 
```

### exec和eval
#### exec 执行
exec(str) ：内置函数，以前是语句，可将用户提供的字符串当做代码执行。</br>
参数：字符串</br>
返回值：无（取决于当做代码执行的字符串是否有返回值）</br>
注意：常用于在线编译器，这样做很危险，不建议使用。</br>
```
>>> exec("print('hello')")
hello
# exec()本身不返回任何东西，只是执行了print，返回值是print的返回值


str_input=input("Place input a command to execute: ")
exec(str_input)

Place input a command to execute: print('hello')
hello
```
#### eval 
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
