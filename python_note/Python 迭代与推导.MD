# Python 迭代与推导
## 迭代
迭代是Python最强大的功能之一，是访问集合元素的一种方式。</br>
常用 for variable in seqence 循环语句来实现迭代。</br>

### 迭代字典
字典项没有具体顺序，所以默认和迭代所有项，而且每次迭代的结果可能不一样。</br>
如果需要保留顺序，可以将字典的数据转换成列表然后进行排序，再对列表进行迭代。</br>

#### 仅根据key进行迭代
```
dict1={'Sharon':25,'Jack':22,'Ben':22,'lulu':3}
for key in dict1:
    print(key,'corresponds to',dict1[key])

Sharon corresponds to 25
Jack corresponds to 22
Ben corresponds to 22
lulu corresponds to 3
```
多层字典迭代
```
>>> def course(IDdict):
...     for classnum in IDdict:
...         student_dict=IDdict[classnum]
...         for name in student_dict:
...             print('class {}:{} ID {}'.format(classnum,name,student_dict[name]))
... 

>>> student_ID
{1: {'Lily': 2, 'Ben': 26}, 2: {'Sharon': 26, 'Lu': 30}, 5: {'Jack': 40}}

>>> course(student_ID)
class 1:Lily ID 2
class 1:Ben ID 26
class 2:Sharon ID 26
class 2:Lu ID 30
class 5:Jack ID 40
>>> 
```

#### 根据项进行迭代
```
for key,value in dict1.items():
    print("{}'s age is {}.".format(key,value))

Sharon's age is 25.
Jack's age is 22.
Ben's age is 22.
lulu's age is 3.
```


### 序列迭代
#### 并行迭代 zip(seq1,seq2,...,seqn)
并行迭代：同时迭代两个序列
```
list2=['Sharon','Jack','Ben','lulu']
list3=[19930226,19960102,19960102,20150905]
# 根据长度迭代
for i in range(len(list2)):
    print("{}'s birthday is {}.".format(list2[i],list3[i]))

Sharon's birthday is 19930226.
Jack's birthday is 19960102.
Ben's birthday is 19960102.
lulu's birthday is 20150905.
```
常用内置函数zip(seq1，seq2)来连接两个在数据值上有一定意义关系的序列，以便同时迭代多个序列。
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

zip(seq1,seq2,seq3)的返回值是多个（由最短的seq决定）根据seq相同index组合的元组，不是一个完整的序列。</br>
zip()返回的是多个格式为“('Sharon', 19930226, 25)”这样的元组，所以zip()的迭代写法必需根据元素值来迭代而不是根据索引。
```
print(zip(list2,list3,list4))
list5=list(zip(list2,list3,list4))    # 连接序列
print(list5)

<zip object at 0xb70ea80c>
[('Sharon', 19930226, 25), ('Jack', 19960102, 22), ('Ben', 19960102, 22), ('lulu', 20150905, 3)]
```
zip(seq1,seq2,...,seqn)也适用于字典，但仅是将字典的key和数组的元素值连起来。
```
>>> dict1={'first':{},'middle':{},'last':{}}
>>> names['Xua','Yu','L']
>>> list(zip(dict1,names))
[('first', 'Xua'), ('middle', 'Yu'), ('last', 'L')]
```

#### 迭代时获取索引 enumerate(seq)
在使用元素值进行迭代的同时获取元素对应的索引，即使用index-value来对序列进行迭代，有点像字典中根据项来迭代一样。</br>
根据index-value来进行迭代，需要用到内置函数enumerate(seq) 枚举。</br>
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


### 和迭代相关的函数：sorted(seq)排序与reversed(seq)反向 
#### sorted(seq) 排序
sorted(seq) ：内置函数，序列元素排序，适用于任何序列或可迭代对象，不修改原序列，返回排序后的新序列。</br>
参数：序列</br>
返回值：序列排序后生成的列表</br>
注意：列表方法 lst.sort(cmp,key,reverse) 会修改原列表，而内置函数sorted(seq)则不会且可用于所有类型序列。</br>
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

#### reversed(seq) 反向
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


## 推导
推导并不是语句，而是表达式。</br>
推导功能强大，常用类似循环和条件语句的形式出现，编写较为简单且可读性高。</br>

### 列表推导
列表推导list comprehension ：从其他列表创建新的列表的方式。</br>
返回值：新列表</br>
#### 用已有列表创建新列表</br>
用[]括起，for前写新列表和旧列表的关系式，关系式后是旧列表的迭代，返回值是一个新列表。</br>
```
# 创建一个由(0,9)每个元素值乘2所得的新列表，然后再乘3生成第二个新列表
list7=[x*2 for x in range(0,10)]
print(list7)
print([x*3 for x in list7])

[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
[0, 6, 12, 18, 24, 30, 36, 42, 48, 54]
```

#### 增加筛选条件
可在迭代for后面增加if条件，对操作对象进行筛选。</br>
注意是对操作对象进行筛选，而不是对结果进行筛选。</br>
```
# 筛选出0到9间可以被3整除的数并计算它们的平方值放在新列表当中
list8=[x*x for x in range(0,10) if x % 3 ==0]
print(list8)

[0, 9, 36, 81]
```

#### 多个for语句合用
多个for语句合用可以同时对多个列表进行操作，得到结果后合成一个新列表。</br>
```
# x分别为[0,1,2]，y分别为[10,11,12]，x和y每个数逐个相互相乘，得长度为len(x)*len(y)的新列表
list9= [x*y for x in range(0,3) for y in range(10,13)]
print(list9)

[0, 0, 0, 10, 11, 12, 20, 22, 24]
```
```
# x和y范围均为0到4，生成 (x,y) 坐标
print([(x,y) for x in range(0,5) for y in range(0,5)])

[(0, 0), (0, 1), (0, 2), (0, 3), (0, 4), (1, 0), (1, 1), (1, 2), (1, 3), (1, 4), (2, 0), (2, 1), (2, 2), (2, 3), (2, 4), (3, 0), (3, 1), (3, 2), (3, 3), (3, 4), (4, 0), (4, 1), (4, 2), (4, 3), (4, 4)]
```
实例</br>
姓名首字母相同的男生和女生配对的高效配对方法。</br>
girls=['alice','ali','sharon','sasa','lily']</br>
boys=['anan','shusi','sahara','ladyfirst']</br>

```
girls=['alice','ali','sharon','sasa','lily']
lettergirls={}
for girl in girls:
    print(lettergirls.setdefault(girl[0],[]))
print(lettergirls)

# 字典方法 setdefault 查询字典lettergirls里是否有当前girl的首字母对应的键。
# 没有则添加一个key为首字母，值为空的项。
#  lettergirls.setdefault(girl[0],[]) 的返回值是添加后的value

[]
[]
[]
[]
[]
{'a': [], 's': [], 'l': []}
```

由于lettergirls.setdefault(girl[0],[]) 的返回值是添加后的value，是个列表，所以可以使用lst.append来进行末尾添加元素。</br>
lst.append是直接修改原列表且没有返回值的，所以返回值value执行value.append(girl)后就会把当前girl加到value的末尾。</br>
```
# 修改循环内的句子为 lettergirls.setdefault(girl[0],[]).append(girl)
girls=['alice','ali','sharon','sasa','lily']
lettergirls={}
for girl in girls:
    lettergirls.setdefault(girl[0],[]).append(girl)
    print(lettergirls)
print(lettergirls)

{'a': ['alice']}
{'a': ['alice', 'ali']}
{'a': ['alice', 'ali'], 's': ['sharon']}
{'a': ['alice', 'ali'], 's': ['sharon', 'sasa']}
{'a': ['alice', 'ali'], 's': ['sharon', 'sasa'], 'l': ['lily']}
{'a': ['alice', 'ali'], 's': ['sharon', 'sasa'], 'l': ['lily']}
```
将男生和女生进行配对，创建了以女生首字母为key的字典后，男生配对女生就不需要所有女生都判断一次，只需要搜索和自己首字母相同的女生就可以了。
```
girls=['alice','ali','sharon','sasa','lily']
boys=['anan','shusi','sahara','ladyfirst']
lettergirls={}
for girl in girls:
    lettergirls.setdefault(girl[0],[]).append(girl)
print([b+'&'+g for b in boys for g in lettergirls[b[0]]])

['anan&alice', 'anan&ali', 'shusi&sharon', 'shusi&sasa', 'sahara&sharon', 'sahara&sasa', 'ladyfirst&lily']
```
这种方法也问题：当多人姓名的首字母相同时，已经被分配的人会继续与后面的人作配对，所以后续还需进一步修复这个问题。


### 字典推导
同列表推导一样，但用{}括起以及for前面有两个用冒号：分隔的key和value的表达式。</br>
格式为 {key表达式：value表达式 for迭代1 for迭代2} 。</br>
返回值：新字典</br>
```
dict_com={i:"{} plus 1 equal {}".format(i,a+1) for i in range(0,4)}
print(dict_com)

{0: '0 plus 1 equal 2557', 1: '1 plus 1 equal 2557', 2: '2 plus 1 equal 2557', 3: '3 plus 1 equal 2557'}
```

