# Python序列：列表
列表：列表是可变的(mutable)，可修改其内容，且提供多种有用的函数。
## list() 类型转换
通过使用list()可以将序列转换为一个列表。(注意：list函数适用于所有类型的序列，而不只是字符串)。
```
>>> help(list)
Help on class list in module builtins:
class list(object)
 |  list(iterable=(), /)
 |  
 |  Built-in mutable sequence.
 |  
 |  If no argument is given, the constructor creates a new empty list.
 |  The argument must be an iterable if specified.
 |  
 |  Methods defined here:
 |  
 |  __add__(self, value, /)
 |      Return self+value.
 |  
 |  __contains__(self, key, /)
 |      Return key in self.
 |  
 |  __delitem__(self, key, /)
 |      Delete self[key].
 |  
 |  __eq__(self, value, /)
 |      Return self==value.
```
```
print(list('hello_world!!!'))  #将字符串转化为序列


['h', 'e', 'l', 'l', 'o', '_', 'w', 'o', 'r', 'l', 'd', '!', '!', '!']
```
## 列表基本操作
除通用操作外，列表可修改，支持元素赋值、删除元素、切片赋值、列表使用。
### 修改列表：元素赋值
```
list1=[0]*3                                                 #初始化长度为3的序列
print("list1=",list1)
list1[2]=input("place input a number: ")                    #给第3个元素赋值
print("list1=",list1)

list1= [0, 0, 0]
place input a number: 3
list1= [0, 0, '3']
```

不能给不存在的元素赋值，所以需要初始化序列长度
```
>>> list2=[0]*2
>>> list2[1]=3
>>> list2
[0, 3]
>>> list2[2]=4
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list assignment index out of range
```

### 删除元素
del语句：删除元素，删除元素后序列长度会减少。
```
num=[1,2,3,4,5,6,7,8,9]
print(num,len(num))
del num[4]                                                  #删除第五个元素
print(num,len(num))

[1, 2, 3, 4, 5, 6, 7, 8, 9] 9
[1, 2, 3, 4, 6, 7, 8, 9] 8
```
### 同时给多个元素赋值：切片赋值
切片赋值：可同时给序列内多个元素赋值，可将切片替换为与自身长度不一致的序列，可在不改变序列原有元素的情况下插入新元素，可用于删除序列中某些元素，支持步长更改。
```
list2=[0]*10
list2[:]=list('helloworld')                                 #替换切片元素
print(list2,len(list2))
list2[5:5]=[" "]                                            #插入元素
print(list2,len(list2))
list2[5:]=[]                                                #删除元素
print(list2,len(list2))

['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd'] 10
['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'] 11
['h', 'e', 'l', 'l', 'o'] 5
```

## 列表方法
方法 object.method(arguments) </br>
详见 https://docs.python.org/3.7/library/functions.html#list
* lst.append(obj)             
将一个对象附加到列表末尾
* lst.clear()                 
清空列表内容，类似于del[:]，会减少序列长度到0
* lst.copy()                  
复制列表
* lst.count(obj)              
计算指定的某元素值在序列中出现了多少次
* lst.extend(seq)             
同时将多个序列添加到某一序列结尾
* lst.index(obj)              
在列表中查找某元素值第一次出现的索引
* lst.insert(index,obj)       
用于将一个对象插入到列表
* lst.pop(index)              
移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
* lst.remove(obj)             
删除第一个为指定值的元素，不返回任何值
* lst.reverse()               
按相反顺序排列列表元素
* lst.sort(cmp,key,reverse)   
在列表内直接对元素进行排序，会修改元素顺序及其对应的index，要保留原列表应先做lst.copy()或直接使用sorted(lst)
    
### append 添加
将一个对象附加到列表末尾</br>
参数：要添加的元素值
```
list2=list(‘hello’)
list2.append("!")                                           #将元素添加到序列末尾
print(list2)

['h', 'e', 'l', 'l', 'o', '!']
```
这个和创建一个空列表再赋值不一样，这个是不需要初始化长度的。</br>

### clear 清空
清空列表内容，类似于del[:]，会减少序列长度到0</br>
参数：无
```
list2=['h', 'e', 'l', 'l', 'o', '!']
list2.clear()                                               #清空列表元素
print(list2,len(list2))

[] 0
```
### copy 复制
复制列表</br>
参数：无
```
a=[1,2,3]
b=a.copy()
print("a= ",a,"b= ",b)                                      #复制整个序列
b[1]=4
print("a= ",a,"b= ",b)                                      #注意原序列的值
c=a                                                         #将c和a序列建立关联
print("a= ",a,"c= ",c)
c[1]=5
print("a= ",a,"c= ",c)                                      #注意a序列值的变化

a=  [1, 2, 3] b=  [1, 2, 3]
a=  [1, 2, 3] b=  [1, 4, 3]
a=  [1, 2, 3] c=  [1, 2, 3]
a=  [1, 5, 3] c=  [1, 5, 3]
```
### count 计数
计算指定的某元素值在序列中出现了多少次</br>
参数：某一元素值
```
list3=list('1234845588892145552555')
print(list3.count('5'))                                    #计算5在序列中一共出现了多少次

8
```
### extend 扩展
同时将多个序列添加到某一序列结尾</br>
参数：元素列表
```
b= [1, 4, 3]
d=[4,5,6]
print("b+d= ",b+d,"b= ",b,"d= ",d)                          #将b序列和d序列连接起来
b.extend(d)                                                 #将d序列的元素扩展到b序列末尾
print("b= ",b,"d= ",d)
e=[7,8,9]
print("e= ",e,"b= ",b)
b=b+e                                                       #同b.extend(e)
print("e= ",e,"b= ",b)

b+d=  [1, 4, 3, 4, 5, 6] b=  [1, 4, 3] d=  [4, 5, 6]
b=  [1, 4, 3, 4, 5, 6] d=  [4, 5, 6]
e=  [7, 8, 9] b=  [1, 4, 3, 4, 5, 6]
e=  [7, 8, 9] b=  [1, 4, 3, 4, 5, 6, 7, 8, 9]
```

### index 索引
在列表中查找某元素值第一次出现的索引</br>
参数：要查找的元素值
```
list3=list('1234845588892145552555')
print(list3.index('8'))                                     #查找8第一次在列表中出现的索引

4
```
### insert 插入
用于将一个对象插入到列表</br>
参数：要插入元素的索引值，要插入的元素值
```
e= [7, 8, 9]
e.insert(2,'hello')                                         #在e的第三个元素处插入hello
print(e)

[7, 8, 'hello', 9]
```
### pop 取出
移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。</br>
pop是唯一修改列表并范围一个非None值的列表方法。</br>
pop为堆栈中的“取出”操作，常与push配对，不过python中没有push，可用append替代。</br>
可选参数：要移除列表元素的索引值，不能超过列表总长度，默认为 index=-1，删除最后一个列表值。</br>
```
x=list('abcdew')
x.pop(3)                                                   #删除列表中的第4个元素
print(x)
x.pop()                                                    #删除列表中最后一个元素
print(x)

['a', 'b', 'c', 'e', 'w']
['a', 'b', 'c', 'e']
```
```
>>> x=list('abcdef')
>>> x								
['a', 'b', 'c', 'd', 'e', 'f']
>>> x.pop()
'f'                                                        #会返回被删除的元素值
>>> x
['a', 'b', 'c', 'd', 'e']
>>> x.pop(2)
'c'
>>> x
['a', 'b', 'd', 'e']
>>> y=[1,2,3,4]						
>>> y.pop()
4
>>> y
[1, 2, 3]
>>> y.append(y.pop())                                       #将刚弹出的值压入
>>> y
[1, 2, 3]
```
### remove 移除
删除第一个为指定值的元素，不返回任何值</br>
参数：某一元素值
```
list3=list('1234845588892145552555')
list3.remove('4')                                           #删除第一个出现元素值为4的元素
print(list3)

['1', '2', '3', '8', '4', '5', '5', '8', '8', '8', '9', '2', '1', '4', '5', '5', '5', '2', '5', '5', '5']
```
### reverse 反向
按相反顺序排列列表元素</br>
参数：无
```
list3=list('1234845588892145552555')
list3.reverse()                                             #反向排序列表中的元素
print(list3)                                                #reverse没有返回值，所以上面一行不能直接写在print()里面

['5', '5', '5', '2', '5', '5', '5', '4', '1', '2', '9', '8', '8', '8', '5', '5', '4', '8', '4', '3', '2', '1']
```
### sort 排序
在列表内直接对元素进行排序，会修改元素顺序及其对应的index，要保留原列表应先做lst.copy()或直接使用sorted(lst)。</br>
参数：</br>
cmp 可选参数, 如果指定了该参数会使用该参数的方法进行排序。</br>
key  主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。</br>
reverse  排序规则，reverse = True 降序， reverse = False 升序（默认）。</br>
默认 cmp=None, key=None, reverse=False</br>
返回值：无</br>

#### 普通排序
```
list3=['5', '5', '5', '2', '5', '5', '5', '1', '2', '9', '8', '8', '8', '5', '5', '4', '8', '4', '3', '2', '1']
list4=list3.copy()                                           #在排序前先复制备份原列表
list3.sort()                                                 #直接在列表内排序
print(list4)
print(list3)

['5', '5', '5', '2', '5', '5', '5', '1', '2', '9', '8', '8', '8', '5', '5', '4', '8', '4', '3', '2', '1']
['1', '1', '2', '2', '2', '3', '4', '4', '5', '5', '5', '5', '5', '5', '5', '5', '8', '8', '8', '8', '9']
```
sorted()的方式可用于任何序列，但总是返回一个列表。
```
list5=sorted(list4)                                          #获得排序后列表但不影响原列表
print(list4)
print(list5)

['5', '5', '5', '2', '5', '5', '5', '1', '2', '9', '8', '8', '8', '5', '5', '4', '8', '4', '3', '2', '1']
['1', '1', '2', '2', '2', '3', '4', '4', '5', '5', '5', '5', '5', '5', '5', '5', '8', '8', '8', '8', '9']
```
```
>>> sorted('hello_world')
['_', 'd', 'e', 'h', 'l', 'l', 'l', 'o', 'o', 'r', 'w']      #返回一个列表
```

#### 高级排序
lst.sort()和sorted(lst)均支持reverse和key参数
* 参数 reverse
```
list4=['5', '5', '5', '2', '5', '5', '5', '1', '2', '9', '8', '8', '8', '5', '5', '4', '8', '4', '3', '2', '1']
list4.sort(reverse=True)                                     #降序直接排列列表
print(list4)

['9', '8', '8', '8', '8', '5', '5', '5', '5', '5', '5', '5', '5', '4', '4', '3', '2', '2', '2', '1', '1']
```
* 参数key
```
week=['Monday','Tuesday','Wednesday','Thursday','Friday','Saterday','Sunday']
week.sort(key=len)                                          #按照元素值长度排序
print(week)

['Monday', 'Friday', 'Sunday', 'Tuesday', 'Thursday', 'Saterday', 'Wednesday']
```

