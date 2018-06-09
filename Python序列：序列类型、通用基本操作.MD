# 序列 sequence
## 数据结构
以某种方式组合起来的数据元素的集合。</br>
Python支持的数据结构基本概念：container</br>
container有三种：序列和映射（字典）、集合set</br>
序列每个元素有编号，映射每个元素有名称（键）</br>
</br>
## Python中的数据结构：sequence
### Python有6种内置序列
* 列表
* 元组
* 字符串
* Unicode字符串
* buffer对象
* xrange对象</br>
最常见为列表和元组。</br>
列表可以更改，尝用于需要中途添加元素的情形；元组不能修改，常用于技术上考虑禁止修改序列的情形。</br>
元组常用于内置函数的返回，常用的情形下几乎都可以用列表来代替元组。</br>
</br>

### 创建列表
列表是最常用的Python数据类型，它可以作为一个方括号内的逗号分隔值出现。</br>
列表的数据项不需要具有相同的类型。</br>
```
year=[2018]                     #单个int作元素要加中括号	
name="Sharon"                   #单个字符串
birth=["Sharon",19930226,25]    #字符串和int作为元素值，注意没有引号的整数才是int
mounth=["Jan","Feb","Mar","Apr","May","June","July","Aug","Sept","Oct","Nov","Dec"]
```


### 通用序列的基本操作
索引(indexing)、分片(slicing)、加(adding)、乘(multiplying)、成员资格检查：检查某个元素是否属于这序列(index)、计算序列长度(len)、找出最大元素和最小元素(min/max)。
#### 索引 indexing</br>
索引：序列中元素的编号，用于访问列表中的元素。</br>
由0开始，使用负数索引时，Python会从右边开始计数(-1指的是序列最右边的元素)。</br>

```
year=[2018];                    #单个int作元素要加中括号	
name="Sharon";
birth=["Sharon",19930226,25];
mounth=["Jan","Feb","Mar","Apr","May","June","July","Aug","Sept","Oct","Nov","Dec"];

print("year[0]: ",year[0]);     #输出第一个元素
print("name[0]:",name[0]);      #Python中没有字符，字符会被当作仅有一个字符的字符串处理
print("name[-1]",name[-1]);     #索引-1指右边起数第一个元素
print("birth[2]:",birth[2]);    #有左边起计算索引由0开始。
print("mounth[1]:",mounth[1]);

num =input("place input a number(1000-9999):")[3]       #输出输入数字的个位数字
print("The single-digit of this number: ",num)
```

执行结果

```
year[0]:  2018
name[0]: S
name[-1] n
birth[2]: 25
mounth[1]: Feb
place input a number(1000-9999):2222
The single-digit of this number:  2
```

#### 切片 slicing
切边：使用分片操作来访问一定范围内元素。</br>
需要两个索引来作为边界，第一个索引的元素是包含在分片内的，而第二个则不包含(这是Python的惯例)。

```
birthmd = input("Place input your birthday (ex:20180607): ")
#切片：截取第5到第8个数字作为变量的值，值会被当作字符处理
print("Your birthday: ",birthmd[4:8])
#注意右边数字不是7是8
print("Your birth-year: ",birthmd[0:-4])
#注意右边数字是-4而不是-5，和索引的-1作对比
```
执行结果
```
Place input your birthday (ex:20180607): 19930226
Your birthday:  0226
Your birth-year:  1993
```
* 简写
```
birthmd = input("Place input your birthday (ex:20180607): ")
print("another way to print your birthday: ",birthmd[-4:])
#如果右边是到序列结尾则可以简略不写
print("another way to print your birth-year: ",birthmd[:3])
#如果左边是序列的开头则可以简略不写
print("Your birth-year and birthday: ",birthmd[:])
#需要整个序列则可以两边都不写，常用于复制序列
```
执行结果
```
Place input your birthday (ex:20180607): 19930226
another way to print your birthday:  0226
another way to print your birth-year:  1993
Your birth-year and birthday:  19930226
```

* 步长
步长：从一个元素移动到另外一个元素的间隔，默认为1，不能为0，为负数时由右边开始提取元素。</br>
```
number=[1,2,3,4,5,6,7,8,9,10]
print("number[0:10:1]= ",number[0:10:1])        #步长为1，由左边开始
print("number[0:10:3]= ",number[0:10:3])        #步长为3，由左边开始
print("number[8:4:-2]= ",number[8:4:-2])        #步长为2，由右边开始
print("number[::2]= ",number[::2])              #包含所有元素或开头结尾时可简写
print("number[::-2]= ",number[::-2])            #输出 10,8,6,4,2
print("number[5::-2]= ",number[5::-2])          #输出 6,4,2，注意索引5对应的是6，开头包括6
print("number[:5:-2]= ",number[:5:-2])          #输出 10,8，注意索引5对应的是6，结尾不包括6
print("number[0:10:-2]= ",number[0:10:-2])      #由0开始回到结尾，由右往左数
```
执行结果
```
number[0:10:1]=  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
number[0:10:3]=  [1, 4, 7, 10]
number[8:4:-2]=  [9, 7]
number[::2]=  [1, 3, 5, 7, 9]
number[::-2]=  [10, 8, 6, 4, 2]
number[5::-2]=  [6, 4, 2]
number[:5:-2]=  [10, 8]
number[0:10:-2]=  []
```


#### 加 adding
加：可以使用加号对两个类型相同的序列进行连接操作</br>
```
hello=["hello"]
world=["world"]
honey=hello+world       #加号连接两个相同类型序列的元素
print(honey)

['hello', 'world']
```

#### 乘 multiplying
乘号：重复某个序列n次并创建新序列
```
number[0:10:1]=  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
hello=["hello"]
world=["world"]
honey=hello+world
print(number*2)		     #乘号：重复序列n次，并创建新序列
print(honey*4)
```
执行结果
```
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
['hello', 'world', 'hello', 'world', 'hello', 'world', 'hello', 'world']
```

* none </br>
Python中，用None表示空。

* 空列表</br>
空列表：[]  除包含[]外，什么都没有的列表。

* 初始化长度序列</br>
```
squence=[0]*10          #创建长度为10的序列
squence2=[None]*9       #用None创建长度为9的序列，注意None开头大写
print(squence)
print(squence2)

[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
[None, None, None, None, None, None, None, None, None]
```


#### 成员资格
in：判断某一值是否在序列当中，是返回true，否返回false
```
>>> per=['r','w','x']
>>> 'x' in per                           #检验x是否是per的成员
True  
>>> 'w' in per
True
>>> 't' in per
False
  
>>> num=[0,1,2,3,4,5,6]                   #元素值类型为int
>>> input("place input a number:") in num
place input a number:6                    #键盘输入值类型为字符串
False                                     #二者类型不匹配所以输出为false
>>> num=['0','1','2','3','4','5','6']     #元素值类型为字符串
>>> input("place input a number:") in num
place input a number:6					
True                                      #修改后类型匹配所以输出为true
>>> 

>>> subject="Hello world!!!"
>>> "llo" in subject                      # in可用于检验某字符串是否为另一字符串的子字符串
True
```

#### 内置函数：长度 len、最大值 max、最小值 min
```
number[0:10:1]=  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
print("the length of number: ",len(number))
print("the max value of number:",max(number))
print("the min value of number:",min(number))
```
执行结果
```
the length of number:  10
the max value of number: 10
the min value of number: 1
```
max和min也可直接使用数作为实参
```
>>> max(2,9,5)
9
>>> min(2,9,5)
2
>>> len(2,9,5)			                  	# len的操作对象是序列
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: len() takes exactly one argument (3 given)
>>> honey="hello world!!"
>>> len(honey)
13
```
