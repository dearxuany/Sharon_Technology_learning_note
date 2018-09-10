# Python IO编程：文件
Python IO编程：</br>
基本上大多数编程语言对于系统的操作都是通过C语言来进行的，即高级语言 - C语言- 物理硬件。</br>
Python上，模块IO是和IO编程相关的常用模块。</br>
Python内置了读写文件的函数，此外还提供了一个fileinput模块，以下描述的和文件有关的操作均属于“同步模式IO编程”。</br>

## 在Python内部对文件进行读写操作
open(filename,mode) ：</br>
open函数用于打开文件，mode用于打开文件之后的操作，mode中的参数可叠加使用。对于一个已存在的文件，如果只是想打开这个文件来看看，则可省略mode参数。如果要打开的文件不在当前文件夹，则需要以文件的绝对路径作为参数。如果发生错误，会报IOError。</br>
返回值：文件对象</br>
注意：两个参数都要带单引号！</br>

```
>>> import io
>>> open('python3_fileTest.txt')
<_io.TextIOWrapper name='python3_fileTest.txt' mode='r' encoding='UTF-8'>
```

### 读取文件内容
读取文件的时候注意一个问题，调用一次read()会将指针放到文件内容结尾。当第二次调用的时候，指针有结尾开始往下读，所以显示的是空。
```
# 读取文件内容，调用read() 方法
>>> f=open('python3_fileTest.txt')
>>> f.read()
'hello!\nworld!\n'
>>> f.read()
''
# 调用seek()将指针指向文件开头
>>> f.seek(0)
0
>>> f.read()
'hello!\nworld!\n'

# 以行为单位返回当前指针所在行的文件内容
>>> f=open('python3_fileTest.txt')
>>> f.readline()
'hello!\n'
>>> f.readline()
'world!\n'

# 以行为单位返回当前指针包括所在行在内的后n行文件内容，0或无参数默认返回所有行，返回值为列表
>>> f=open('python3_fileTest.txt')
>>> f.readlines(1)
['hello!\n']
>>> f.readlines(1)
['world!\n']
>>> f.readlines()
['hello!\n', 'world!\n']
```

### 文件写入
文件只写入有两个mode，w覆盖和a追加，单独用这两个只能写入不能读，w和a均可新建文件。
```
# 只有在open的时候设置了mode为w才能调用write方法
>>> f=open('python3_fileTest.txt')
>>> f.write('hello!')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
io.UnsupportedOperation: not writable

>>> f=open('python3_fileTest.txt','w')
>>> f.write('hello!')
6

# 因为open的时候没有加r，所以read方法无法调用
>>> f.read()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
io.UnsupportedOperation: not readable

# 设置的mode是w，w是从文件开头开始写入（截取），所以原来的数据没有了
>>> f=open('python3_fileTest.txt')
>>> f.read()
'hello!'

# 设置mode为a，则写入位置为文件结尾，不会删除原有内容
>>> f=open('python3_fileTest.txt','a')
>>> f.write('world!')
6
>>> f=open('python3_fileTest.txt')
>>> f.read()
'hello!\nworld!'
```
### 关闭文件
当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用close()方法时，操作系统才保证把没有写入的数据全部写入磁盘。忘记调用close()的后果是数据可能只写了一部分到磁盘，剩下的丢失了。
通常和with ... as ... 语句配合使用，with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。</br>
详情关注：https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/

```
# 关闭文件对象，关闭后即无法对对象调用任何方法
>>> f.close()
>>> f.write('hello!')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: I/O operation on closed file.

# 无论文件是否被成功打开，都需要执行关闭操作
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()

# 为减少代码，可用with改写，with自带关闭功能，所以不需要调用close
with open('/path/to/file', 'r') as f:
    print(f.read())
```

### 文件读写
r+ ：可读可写，覆盖或追加由指针位置决定，若文件不存在，报错；</br>
w+ ：可读可写，始终从头开始写（覆盖），只有open一执行马上清空文件的内容，若文件不存在，创建；</br>
a+ ：可读可写，始终从内容结尾开始写（追加），若文件不存在，创建；</br>
建议使用r和a、r+、a+，使用r+的时候要注意指针在文件中的位置，a+的写操作指针永远在文件内容最后。</br>

```
# 使用r+，文件不存在报错
>>> f=open('python3_fileTest2.txt','r+')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileNotFoundError: [Errno 2] No such file or directory: 'python3_fileTest2.txt'

# 使用w+，结果发现什么都没有写入，但文件成功创建
>>> f=open('python3_fileTest2.txt','w+')
>>> f.write('Hello world!')
12
>>> f.read()
''
>>> f.close()

# 重新打开，还是发现无法写入，大概是因为第一次使用r+之后没有正常关闭导致的
>>> f=open('python3_fileTest2.txt','w+')
>>> f.read()
''
```
```
# 使用w+ 的正确流程，可是不建议用这个
>>> f=open('python3_fileTest2.txt','w+')
>>> f.write('Hello world!')
12
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
'Hello world!'
>>> f.close()

# 使用a+ 追加内容
>>> f=open('python3_fileTest2.txt','a+')
>>> f.write('Good morning!')
13
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
'Hello world!\nGood morning!'
>>> f.close()
```
读写应该注意的一些关于指针的小问题：
```
>>> f=open('python3_fileTest3.txt','a+')
# a+ 参数，指针在内容末尾，所以就算有内容，也读不到任何东西，a+模式write指针始终指向
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
'a testing'
>>> f.seek(0)
0
>>> f.write('b')
1
>>> f.seek(0)
0
>>> f.read()
'a testingb'
>>> f.close()
```
```
# w+ 参数，感觉有点危险，因为open()函数一执行，文件里的数据就会被全部清空，不管你之后有没有写入和close
>>> f=open('python3_fileTest3.txt','w+')
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
''
```
```
# r+ 参数，使用的时候要特别注意指针位置，可使用tell()方法来返回当前位置
>>> f=open('python3_fileTest2.txt','r+')
>>> f.write('hello!')
6
>>> f.read()
''
>>> f.write('world!')
6
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
'hello!world!'
>>> f.write('good moring!')
12
>>> f.read()
''
>>> f.seek(0)
0
>>> f.read()
'hello!world!good moring!'
>>> f.seek(0)
0
>>> f.write('python')
6
>>> f.seek(0)
0
>>> f.read()
'pythonworld!good moring!'
```

### 配合linux管道命令重定向输出
linux管道命令：</br>
管道将一个命令的标准输出连接到下一个命令的标准输入，整个过程不影响提供数据来源的文件本身。</br>
</br>
Python和管道命令有关的模块为sys ，这个模块里有很多与操作系统相关的函数。Python可以通过这个模块和操作系统进行交互。</br>
```
# SHELL管道命令：计算标准字典文件里的词汇数
$ cat /usr/share/dict/words|python3 python3_u11_fileCtrl_wordsCount.py 
Wordscount: 479828
# 计算python3_fileTest3.txt中的词汇数
$ cat ./python3_fileTest3.txt |python3 ../python3_u11_fileCtrl_wordsCount.py 
Wordscount: 31
```
python脚本
```
#! /usr/bin/python3

import sys

# stdin 可直接调用read()方法
text=sys.stdin.read()

# str方法split,默认分隔符号为空格
words=text.split()
# list函数
wordsCount=len(words)

print('Wordscount:',wordsCount)
```
