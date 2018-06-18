# Python映射：字典
## Python 数据结构
Python支持的数据结构基本概念：container</br>
container有三种：序列、映射（字典）、集合set</br>
序列每个元素有编号，映射每个元素有名称（键）。</br>

* 映射 mapping</br>
可以通过名称来访问各个值的数据结构</br>

* 字典 dictionary</br>
Python中唯一内置的映射类型，值不按顺序排列，而存储在键下。</br>
目的：为方便找到特定的单词（键 key），以获悉其定义（值 value）。</br>
键可以是数字、字符串、元组。</br>

## 创建字典
项 item ：键-值的组合。</br>
键与值间用冒号：分隔，项之间用逗号，分隔，字典用花括号{}括起。空字典用{ }表示。</br>
键必须是唯一的，但是值不需要。同一字典中的键不需要是相同类型。</br>
```
>>> dictionary={"Sharon":25,"Jack":22,"Ben":22}			#创建一个新字典，注意有等号
>>> dictionary										#查看整个字典
{'Sharon': 25, 'Jack': 22, 'Ben': 22}
>>> dictionary["Jack"]								#查看某个键对应的值
22
```
键是不但不能重复，还区分大小写。
```
>>> dictt={'lulu':22,'Tom':23}
>>> dictt
{'lulu': 22, 'Tom': 23}
>>> dictt['Lulu']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'Lulu'
>>> dictt['lulu']
22
```

## dict()函数 转换为字典
函数dict()：可以把其他已存在的映射或有item结构的序列转换为字典。
```
items1=[('Sharon',25),('Ben',22),('lulu',21)]           #注意item的写法
print('items1: ',items1)
dict1=dict(items1)                      #将序列转换为字典
print('dict1:',dict1)
print('dict1["lulu"]:',dict1['lulu'])

items1:  [('Sharon', 25), ('Ben', 22), ('lulu', 21)]
dict1: {'Sharon': 25, 'Ben': 22, 'lulu': 21}
dict1["lulu"]: 21
```
dict()内不加任何参数会生成空字典。
```
>>> dicti=dict()
>>> dicti
{}
```
序列如果没有item的结构是无法转换成字典的。
```
>>> a=['a','b','v','e']
>>> dict(a)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: dictionary update sequence element #0 has length 1; 2 is required
```


## 字典的基本操作
### 字典的基本操作
字典与序列有相似的基本操作，但是有略微的区别。
* dict[key] 查询键的值
* dict[key]=value 将value关联给key
* del dict[key] 删除key对应的item
* key in dict 成员资格检查，判断依据是key，返回布尔型
* len(dict) 查询字典dict的item数

```
dict1= {'Sharon': 25, 'Ben': 22, 'lulu': 21}
print('dict1:',dict1)
print('dict1["lulu"]:',dict1['lulu'])   #查询键为lulu的值

len(dict1)              #输出dict1的项数

dict1['Tom']=33         #将键Tom关联到值33，注意这是不需要额外的方法的
print('dict1: ',dict1)

del dict1['Ben']       	 #删除键Ben的项
print('Ben' in dict1)   	#检查'Ben'是否是dict1的键
print('Tom' in dict1)

dict1: {'Sharon': 25, 'Ben': 22, 'lulu': 21}
dict1["lulu"]: 21
dict1:  {'Sharon': 25, 'Ben': 22, 'lulu': 21, 'Tom': 33}
False
True
```

### 字典与序列基本操作区别
* 键的类型：字典中的键可以为不可改变的类型，如浮点型（实数）、元组、字符串。
* 新加项：字典可以直接以dict[key]=value来添加原本字典中没有的项，不需要初始化长度后赋值，也不需要像列表一样需要用append()来添加新值。
* 成员资格：字典判断的依据是key，而序列判断的依据是value。数据结构越大，字典成员资格检查效率越高。

### 字典创建简单数据库及查询
dict people </br>
行name:items1</br>
列items1 age:age_number, birth:birthday</br>
```
people={
        'Sharon':{
        'age':25,
        'birthday':'0226'
        },
        'Tom':{
        'age':22,
        'birthday':'0708'
        },
        'Lily':{
        'age':23,
        'birthday':'0125'
        }
}

print(people)
print("people['Sharon']['age']",people['Sharon']['age'])
print("people['Lily']['birthday']",people['Lily']['birthday'])
print("people['Sharon']",people['Sharon'])
print("people['age']",people['age'])		#这句是错误的，不能不按顺序查询


{'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}, 'Lily': {'age': 23, 'birthday': '0125'}}
people['Sharon']['age'] 25
people['Lily']['birthday'] 0125
people['Sharon'] {'age': 25, 'birthday': '0226'}
Traceback (most recent call last):
  File "python3_u4_mapping_dictionary.py", line 42, in <module>
    print("people['age']",people['age'])
KeyError: 'age'
```

### 字典的格式化字符串
* str.format_map(dict) 配合字典格式化字符串
```
dict1={'Sharon': 25, 'lulu': 21, 'Tom': 33}
print("lulu is {lulu} yearsold.".format_map(dict1))     
#格式化字符串,注意{}内不需要再加引号
name=input('Place input your name: ')
print("{} is {} yearsold.".format(name,dict1[name]))    #注意和上面的写法是不样的

lulu is 21 yearsold.
Place input your name: Sharon
Sharon is 25 yearsold.
```


* %格式化字符串
在转换说明符(%)后面加上一个圆括号，圆括号里面是字典的键，接着后面是其他的说明元素。
	（这个方法没有试验成功...）



## 字典方法
### clear 清除
dict.clear() 删除字典中所有元素，字典被清空为空字典</br>
参数：无</br>
返回值：无</br>
```
dict2={'a':1,'b':2,'c':3}
print("dict2=",dict2)
dict2.clear()   #删除字典中所有项
print("dict2=",dict2)

dict2= {'a': 1, 'b': 2, 'c': 3}
dict2= {}
```
### copy 浅复制 和 deepcopy 深复制
* copy 浅复制</br>
dict.copy()：浅复制，原件中值变更会影响副本，副本中值的变更不会影响原件，但无论是原件还是副本做删除和添加元素操作都会同时影响原件和副本。</br>
参数：无</br>
返回值：无</br>

* deepcopy 深复制</br>
copy.deepcopy(dict)：深复制副本可以作为一个独立的原件，和原件各自独立，不相互影响。</br>
参数：无</br>
返回值：无</br>
```
import copy                     #必须要加这句，否则deepcopy会出错，调用deepcopy(),deepcopy()在模块copy中
dict2={'a':[1,2,3,4],'b':5}
print('dict2=',dict2)
dict3=dict2.copy()              	#浅复制
dict4=copy.deepcopy(dict2)      ·#深复制

dict2['b']='c' 			#修改原件值
dict2['a'].remove(4)   	#删除原件值
dict2['a'].append('e')  	#添加原件值
print('dict2=',dict2)
print('dict3=',dict3)
print('dict4=',dict4)

dict2= {'a': [1, 2, 3, 4], 'b': 5}		#原字典
dict2= {'a': [1, 2, 3, 'e'], 'b': 'c'}		#原字典修改，删除4，添加e
dict3= {'a': [1, 2, 3, 'e'], 'b': 5}		#浅复制副本，跟随原字典变化
dict4= {'a': [1, 2, 3, 4], 'b': 5}		#深复制，完全不影响
```
```
import copy                     #必须要加这句，否则deepcopy会出错
dict2={'a':[1,2,3,4],'b':5}
print('dict2=',dict2)
dict3=dict2.copy()              #浅复制
dict4=copy.deepcopy(dict2)      #深复制

dict3['b']='c'  #修改浅复制值
dict3['a'].remove(4)    #删除浅复制值
dict3['a'].append('e')  #添加浅复制值
print('dict2=',dict2)
print('dict3=',dict3)
print('dict4=',dict4)

dict2= {'a': [1, 2, 3, 4], 'b': 5}		#原字典
dict2= {'a': [1, 2, 3, 'e'], 'b': 5}		#浅复制副本修改，原字典值不改变；浅复制副本添加或删除，改变原字典
dict3= {'a': [1, 2, 3, 'e'], 'b': 'c'}		#浅复制副本修改，删除4，添加e
dict4= {'a': [1, 2, 3, 4], 'b': 5}		#深复制不影响
```
```
import copy                     #必须要加这句，否则deepcopy会出错
dict2={'a':[1,2,3,4],'b':5}
print('dict2=',dict2)
dict3=dict2.copy()              #浅复制
dict4=copy.deepcopy(dict2)      #深复制

dict4['b']='c'  #修改深复制值
dict4['a'].remove(4)    #删除深复制值
dict4['a'].append('e')  #添加深复制值
print('dict2=',dict2)
print('dict3=',dict3)
print('dict4=',dict4)

dict2= {'a': [1, 2, 3, 4], 'b': 5}
dict2= {'a': [1, 2, 3, 4], 'b': 5}
dict3= {'a': [1, 2, 3, 4], 'b': 5}
dict4= {'a': [1, 2, 3, 'e'], 'b': 'c'}		#深复制副本修改，仅影响深复制副本自身
```
详情见</br> https://docs.python.org/3.7/library/copy.html</br>
https://www.programiz.com/python-programming/shallow-deep-copy


### fromkeys 新建包含指定键且值为None的字典
dict.fromkeys(key)：新建包含指定键且值为None的字典</br>
参数：无</br>
返回值：返回新建的字典</br>
```
dict5={}.fromkeys(['a','b'])	#语法上需要在fromkeys前面加{}
dict6=dict.fromkeys(['c','d'])	#dict是所有字典的类型，所以可以这么用
dict7=dict.fromkeys(['a','b'],'c')
print(dict5,dict6,dict7)

{'a': None, 'b': None} {'c': None, 'd': None} {'a': 'c', 'b': 'c'}
```
### get
dict.get(key)：用get来访问字典中没有的值时不会产生错误，而是返回一个None（可修改）。常用于数据库查询，输入不合法的处理。</br>
参数：键</br>
返回值：None或指定值，键对应的值</br>
```
dict7={'a': 'c', 'b': 'c'}
key7=input('Place input a key: ')
print(dict7.get(key7))
print(dict7.get(key7,'absent'))		#指定返回值

Place input a key: oo
None
absent

Place input a key: a		#含有该键正常返回
c
c
```

### items
dict.items()：返回一个包含所有字典项的序列，形式为(key,value)，顺序不确定。</br>
参数：无</br>
返回值：列表</br>
```
people={'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}, 'Lily': {'age': 23, 'birthday': '0125'}}
print(people.items())

dict_items([('Sharon', {'age': 25, 'birthday': '0226'}), ('Tom', {'age': 22, 'birthday': '0708'}), ('Lily', {'age': 23, 'birthday': '0125'})])		#但是value里的项是没有变化的，依然是字典的表示形式
```
### keys
dict.keys()：返回字典中的键</br>
参数：无</br>
返回值：字典视图</br>
```
people={'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}, 'Lily': {'age': 23, 'birthday': '0125'}}
print(people.keys())

dict_keys(['Sharon', 'Tom', 'Lily'])	# value中的keys不会被返回
```
### values
dict.values()：返回字典中的value</br>
参数：无</br>
返回值：字典视图</br>
```
people={'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}, 'Lily': {'age': 23, 'birthday': '0125'}}
print(people.values())

dict_values([{'age': 25, 'birthday': '0226'}, {'age': 22, 'birthday': '0708'}, {'age': 23, 'birthday': '0125'}])
```


### pop查询并删除项
dict.pop(key)：显示key对应的值，并删除整个项。</br>
参数：键</br>
返回值：该键对应的值</br>
```
dict7={'a': 'c', 'b': 'c'}
print(dict7.pop('a'))
print('dict7=',dict7)

dict7= {'b': 'c'}
```

### popitem 随机抛出项并删除
dict.popitem()：随机抛出一个字典项并删除它。</br>
参数：无</br>
返回值：项</br>
```
people={'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}, 'Lily': {'age': 23, 'birthday': '0125'}}
print(people.popitem())
print(people)

('Lily', {'age': 23, 'birthday': '0125'})
{'Sharon': {'age': 25, 'birthday': '0226'}, 'Tom': {'age': 22, 'birthday': '0708'}}
```
### setdefault 查询添加
dict.setdefault(key)：在字典中查找key对应的值，键存在则返回值，键不存在则在字典中添加新项（key:None），项的值可以指定。</br>
```
dict7={'a': 'c', 'b': 'c'}
key7=input('Place input a key: ')
print(dict7.setdefault(key7))
print(dict7.setdefault('k','absent'))
print(dict7)

Place input a key: g
None
absent
{'a': 'c', 'b': 'c', 'g': None, 'k': 'absent'}
```

### update 
dict1.update(dict2)：将dict2里的项添加到dict1，如果字典中有相同的键则替换该键的value。</br>
参数：字典</br>
返回：无</br>
```
dict5={'a': None, 'b': None} 
dict6={'c': None, 'd': None}
dict5.update(dict6)
print(dict5,dict6)
 
{'a': None, 'b': None, 'c': None, 'd': None} {'c': None, 'd': None}
```
