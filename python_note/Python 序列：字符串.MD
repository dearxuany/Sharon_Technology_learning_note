# Python序列：字符串
支持各种标准序列操作，不支持元素赋值和切片赋值。</br>
不可变更，可以使用引号('或")来创建字符串。</br>
Python中没有字符，会把字符看做长度为1的字符串。</br>
在Python2中，普通字符串是以8位ASCII码进行存储的。</br>
在Python3中，所有的字符串都是Unicode字符串。</br>


## 创建字符串
### 单行字符串
使用引号('或")来创建字符串
```
>>> var1='hello'
>>> varr="world"
>>> var1+varr
'helloworld'
>>> var1[1:3]       #查看字符串的值
'el'
>>> var1[:1]+"i"
'hi'                #新的一个字符串
>>> var1
'hello'             #原字符串值不变
```

使用\来换行输入
```
var2='hello\        # \可换行输入
... earth'
>>> var2
'helloearth'        #注意\是不影响字符串本身
```

### 建立多行文本
使用单引号’’’或双引号”””来建立跨行文本
```
>>> s='''                 # ’’’建立跨行文本
... Good
... morning
... !!!'''
>>> s
'\nGood\nmorning\n!!!'    #注意换行符\n，’’’会影响字符串的值
```
```
>>> s2="""                # ”””建立跨行文本
... Good
... afternoon
... !!!
... """                   #注意”””的位置会影响\n
>>> s2
'\nGood\nafternoon\n!!!\n'
```

## str() 类型转换
通过使用str() 可以将序列转换为一个字符串。
```
>>> a=18
>>> b=adg                 #没有加引号所以出现错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'adg' is not defined
>>> b='adg'
>>> c=b+a                 #b和a类型不一样，所以出现错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: cannot concatenate 'str' and 'int' objects
>>> c=b+str(a)            #将a转回成字符串
>>> c
'adg18'
```
```
>>> num=[1,2,3]
>>> num
[1, 2, 3]
>>> str(num)
'[1, 2, 3]'               #整个序列转换为了1个字符串
>>> str（num[1])          #不能对单个元素操作
  File "<stdin>", line 1
    str（num[1]）
          ^
SyntaxError: invalid character in identifier
>>> num                   #不影响原序列本身
[1, 2, 3]
```
```
num=[1,2,3,4]
print(num)
str1=str(num)             #str将序列转换为字符串
print(str1)

[1, 2, 3, 4]
[1, 2, 3, 4]              #脚本运行结果没有显示出是字符串但类型已变换
```


## 更新或删除字符串
字符串无法被更改，不支持元素赋值和切片赋值。
```
>>> var1='hello'
>>> var1[2]=f
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'f' is not defined
```
无法删字符串中的某个值，但是可以用del把字符串整个删除。
```
>>> del var1(2)
  File "<stdin>", line 1
SyntaxError: can't delete function call
>>> del var1
>>> var1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'var1' is not defined
```


## 字符串基本操作
支持序列通用基本操作</br>
详见
[Python序列：序列类型、通用基本操作](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/Python%20%E5%BA%8F%E5%88%97%EF%BC%9A%E5%BA%8F%E5%88%97%E7%B1%BB%E5%9E%8B%E3%80%81%E9%80%9A%E7%94%A8%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C.MD)


## 字符串方法
详见 Python的说明文档 https://docs.python.org/3.7/library/string.html
* str.center(width[, fillchar])</br>
在两边添加填充字符，默认为空格，使字符串居中。
* str.find(str, beg=0, end=len(string))</br>
检测字符串中是否包含子字符串，如果有则返回查找值对应第一个出现的元素的索引，反则返回-1。
* str.join(seq)</br>
将指定字符串插入到另外一个序列中，与split作用相反。
* str.lower()</br>
将字符串转换为小写。
* str.upper()</br>
将字符串转换为大写。
* str.title()</br>
将字符串转换为词首大写。
* str.replace(obj1,obj2)</br>
用字符串obj2替换字符串obj1。
* str.split(str="", num=string.count(str))</br>
通过指定分隔符对字符串进行拆分，如果参数num 有指定值，则仅分隔 num 个子字符串。作用上，与join相反。
* str.strip([chars])</br>
删除字符串前后的某些字符，默认为空格，返回删除后结果。
* str.maketrans(intab, outtab，[char])</br>
创建字符串的转换表，指出不同Unicode码点之间的转换关系。操作对象是单个字符，translate会用到。
* str.translate(table)</br>
根据转换表，对字符串中的字符进行转换或删除，可同时转换多个字符。
* str.endswith(obj[,start[,end]])</br>
判断字符串str是否为chars结束，是返回True，否返回False。
* str.startwith(obj)</br>
判断字符串str是否以obj开头，返回布尔型。
* 判断字符串是否满足特定性质</br>
以is打头的方法：判断字符串是否满足特定性质，满足返回True，否则返回False。

### center 居中
str.center(width[, fillchar]) ：在两边添加填充字符，默认为空格，使字符串居中。</br>
参数：width 宽度、fillchar 填充字符</br>
返回值：填充后的字符串</br>
```
str2="hello_world"
str3=str2.center(20)              #空格填充，宽度20,字符串居中
str4=str2.center(20,'*')          #用*作为填充字符
print(str2)
print(str3)
print(str4)
print("len of str4:",len(str4))

hello_world
    hello_world     
****hello_world*****
len of str4: 20                   #整个字符串长度为20
```

```
>>> "hello".center(21,'~')
'~~~~~~~~hello~~~~~~~~'           #注意返回值是居中后的字符串
```

### find查找
str.find(str, beg=0, end=len(string)) ：检测字符串中是否包含子字符串，如果有则返回查找值对应第一个出现的元素的索引，反则返回-1。</br>
参数：beg 开始索引，默认为0。end 结束索引，默认为字符串的长度。起点包含在搜索范围，而终点不包含。</br>
返回值：有，返回第一个找到的元素的索引；无，返回-1。注意返回值并非布尔型。</br>
```
>>> "hello_world".find('llo')
2                                 #含llo返回元素索引
>>> "hello_world".find('hi')
-1                                #不含hi返回-1
```
```
str2="hello_world"
print(str2.find('llo'))                 #默认查找整个字符串
print(str2.find('llo',2,8))             #设置起点索引为2，结束为8
print(str2.find('llo',5,len(str2)))     #llo在搜索范围之外

2
2
-1
```
### join 合并（参加）
str.join(seq)：将指定字符串插入到另外一个序列中，与split作用相反。</br>
参数：被插入的序列</br>
返回值：插入字符串后的序列</br>
```
str2="hello_world"
list1=list(str2)
s='~'
print(s.join(list1))                                      #将～插入到list1
print(list1)
print(s.join(str2))                                       #参数可以是字符串

h~e~l~l~o~_~w~o~r~l~d
['h', 'e', 'l', 'l', 'o', '_', 'w', 'o', 'r', 'l', 'd']   #不影响原序列
h~e~l~l~o~_~w~o~r~l~d
```
```
>>> list1=[1,2,3]
>>> s='~'
>>> s.join(list1)                                         #元素类型必须都是字符串
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sequence item 0: expected str instance, int found
>>> 
```

### lower 转换为小写 upper 转换为大写
str.lower() ：将字符串转换为小写</br>
str.upper() ：将字符串转换为大写</br>
参数：无</br>
返回值：返回转换后的字符串</br>
```
str2="hello_world"
str5="GOOD MORNING" 
print(str5.lower())           #将字符串转换为小写
print(str5.title())           #将字符串转换为词首大写
print(str5)                   #转换大小写不影响原字符串
print(str2.upper())           #将字符串转换为大写

good morning
Good Morning
GOOD MORNING
HELLO_WORLD
```

str.title() ：将字符串转换为词首大写</br>
参数：无</br>
返回值：返回转换后的字符串</br>
局限：难以确定单词边界，容易发生判断错误</br>

### replace 替代
str.replace(obj1,obj2) ：用字符串obj2替换字符串obj1</br>
参数：obj1 被替代的字符串，obj2 替代字符串</br>
返回值：替代后的字符串</br>
```
str5="GOOD MORNING"
print(str5.replace('MORNING','AFTERNOON'))      #替换字符串

GOOD AFTERNOON
```

### split 拆分
str.split(str="", num=string.count(str)) ：通过指定分隔符对字符串进行拆分，如果参数num 有指定值，则仅分隔 num 个子字符串。作用上，与join相反。</br>
参数：</br>
str    指定的分隔符，没有指定时，默认为所有的空字符，包括空格、换行(\n)、制表符(\t)等。</br>
num    分割次数</br>
```
str6=’h~e~l~l~o~_~w~o~r~l~d’
print(str6.split('~'))                          #拆分字符串 

['h', 'e', 'l', 'l', 'o', '_', 'w', 'o', 'r', 'l', 'd']
```
```
>>> str='How are you ?'
>>> str.split()
['How', 'are', 'you', '?']
>>> str1='hello_world'
>>> str1.split()
['hello_world']
>>> str1='hello\nworld'
>>> str1.split()
['hello', 'world']
```
### strip 删除（剥夺）
str.strip([chars]) ：删除字符串前后的某些字符，默认为空格，返回删除后结果。</br>
参数：指定要删除的字符</br>
返回：删除后结果</br>
```
str7='hello~world'.center(21,'~') 
print(str7)
print(str7.strip('~'))                  #删除前后的～，注意中间不会被删除
str8='hello world'.center(21)
print(str8)
print(str8.strip())

~~~~~hello~world~~~~~
hello~world
     hello world     
hello world
```

### maketrans 创建转换表
str.maketrans(intab, outtab，[char]) ：创建字符串的转换表，指出不同Unicode码点之间的转换关系。操作对象是单个字符，translate会用到。</br>
参数：</br>
intab 原字符串将要被转换的字符；</br>
outtab 将要转换成的字符；</br>
char 可选，将要被删除的字符。</br>
返回值：无</br>
```
str7=’~~~~~hello~world~~~~~’
tab=str7.maketrans('lo','!0')             #创建转换表
print(tab)
str8=’     hello world     ’
tab2=str8.maketrans('lo','!0',' ')        #转换同时给出要删除的字符
print(tab2)

{108: 33, 111: 48}                        #转换表，指出Unicode码点的转换关系
{108: 33, 111: 48, 32: None}
```

### translate 转换（翻译）
str.translate(table) ：根据转换表，对字符串中的字符进行转换或删除，可同时转换多个字符。</br>
参数：table 用maketrans制成的转换表</br>
返回值：转换后的字符串</br>
```
str7=’~~~~~hello~world~~~~~’
tab=str7.maketrans('lo','!0')             #创建转换表
str8=’     hello world     ’
tab2=str8.maketrans('lo','!0',' ')        #转换同时给出要删除的字符
print(str7.translate(tab))                #字符转换
print(str8.translate(tab2))               #字符转换及删除

~~~~~he!!0~w0r!d~~~~~
he!!0w0r!d
```

### endswitch 判断结尾
str.endswitch(chars[,start[,end]])：判断字符串str是否为chars结束，是返回True，否返回False。</br>
参数：要查询的字符串，起始位置。</br>
返回：布尔型</br>
```
str2= hello_world
print(str2.endswith('world'))             #判断结尾
print(len(str2))
print(str2.endswith('orl',6,10))          #限制范围
print(str2.endswith('orl'))
print(str2.endswith('hello',7,11))

True
11
True
False
False
```

### startswith 判断开头
str.startswitch(chars[,start[,end]])：判断字符串str是否为chars开始，是返回True，否返回False。</br>
参数：要查询的字符串，起始位置。</br>
返回：布尔型</br>
```
str2= hello_world
print(str2.startswith('hello'))         #判断开头
print(str2.startswith('hello',0,4))
print(str2.startswith('hello',0,5))

True
False
True
```


### 判断字符串是否满足特定性质
以is打头的方法：判断字符串是否满足特定性质，满足返回True，否则返回False。</br>
参数：无</br>
返回值：布尔型</br>
* str.isdecimal()   检查字符串是否只包含十进制字符，只存在于unicode对象。
* str.isalnum()     检测字符串是否由字母和数字组成。
* str.isalpha()     检测字符串是否只由字母组成。
* str.isdigit()     检测字符串是否只由数字组成。
* str.isnumeric()   检测字符串是否只由数字组成，只针对unicode对象。
* str.islower()     检测字符串是否由小写字母组成。
* str.isupper()     检测字符串中所有的字母是否都为大写。
* str.istitle()     检测字符串中所有的单词拼写首字母是否为大写，且其他字母为小写。
* str.isspace()     检测字符串是否只由空白字符组成。
* str.isprintable() 检验字符串中所有字符是否都是可打印字符(in repr())或字符串为空。 

## 扩展
* Unicode字符集中“Other” “Separator”类别的字符为不可打印的字符（但不包括ASCII码中的空格（0x20））。可用于判断转义字符。
* Unicode字符串则存储为16位unicode字符串，这样能够表示更多的字符集。使用的语法是在字符串前面加上前缀 u。
* SCII码中第0～32号及第127号是控制字符；第33～126号是可打印字符，其中第48～57号为0～9十个阿拉伯数字；65～90号为26个大写英文字母，97～122号为26个小写英文字母。
