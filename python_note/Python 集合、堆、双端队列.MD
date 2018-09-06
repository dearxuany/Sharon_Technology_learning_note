# Python 集合、堆、双端队列
集合 set、堆 heap、双端队列deque 是除序列和映射外的另外一些常用数据结构。</br>
Python中有现成的集合set类型；</br>
Python中没有独立的堆类型，必须使用列表来表示堆本身，但是包含一个有堆操作函数的模块heapq；</br>
Pyhon 中有双端队列 deque 和其他一些 collection 类型，它们被包含在模块 collections 当中。</br>

## 集合
这里的集合就是指初中数学学的那个集合。</br>
集合 set ：</br>
用花括号{}括起一系列元素，元素间由逗号分隔，每个元素排列顺序不确定（每次输出显示的顺序不一样），集合会自动过滤掉重复的元素。集合主要用于成员资格检查。</br>

### 创建集合
使用set()创建</br>
```
>>> set(range(20))
{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19}
```

直接创建
```
>>> {5,9,5,8,6,63,5}
{5, 6, 8, 9, 63}
>>> j={2,3,9}
>>> type(j)
<class 'set'>
# 不能使用以下方法创建空集合
>>> type({})
<class 'dict'>
```

可以让变量关联到数字，然后再用该变量作为集合元素，也可用字符串作为集合元素的。
```
>>> a=3
>>> {a,7}
{3, 7}
>>> d=4
>>> {a,d}
{3, 4}
>>> 
>>> {'a','b',1}
{1, 'b', 'a'}

```
集合set是可变的，但是集合的元素不能是可变的，即是集合中不能包含集合。
```
>>> c={1,4,6,{5,7},10}
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'set'
>>>
```
要让集合包含集合必须使用set的另外一种集合类型frozenset，它表示不可变集合。
```
>>> j
{9, 2, 3}
>>> k
{8, 3, 6}
>>> j.add(frozenset(k))
>>> j
{9, 2, 3, frozenset({8, 3, 6})}
```
### 集合基本操作
#### 集合关系
一般关系：
```
>>> j={2,3,9}
>>> k={3,6,8}

# 并集
>>> j.union(k)
{2, 3, 6, 8, 9}
>>> j | k
{2, 3, 6, 8, 9}
>>> 

# 交集
>>> j.intersection(k)
{3}
>>> j & k
{3}

# 差集
>>> j.difference(k)
{9, 2}
>>> k.difference(j)
{8, 6}
>>> j-k
{9, 2}
>>> k-j
{8, 6}

# 对称差集 = 并集-交集
>>> j.symmetric_difference(k)
{2, 6, 8, 9}
>>> j ^ k
{2, 6, 8, 9}
```
成员资格检验：
```
>>> a={6,9,5,4,7}
>>> b={4,5,6}

# 子集判断
>>> b.issubset(a)
True
>>> b <= a
True

# 父集判断
>>> a.issuperset(b)
True
>>> a >= b
True
```

#### 元素变更
```
>>> b={4,5,6}

# 添加一个新元素
>>> b.add(9)
>>> b
{9, 4, 5, 6}
# 添加多个新元素
>>> b.update([2,6,7])
>>> b
{2, 4, 5, 6, 7, 9}

# 删除元素
>>> b.remove(5)
>>> b
{2, 4, 6, 7, 9}
```

## 堆
堆heap 是一种特殊数据结构，python中用列表来表示堆对象，利用模块heapq 中的堆函数对堆对象进行操作。</br>
堆总是满足下列性（堆特征heap property） ：</br>
1.堆中某个节点的值总是不大于或不小于其父节点的值；</br>
2.堆总是一棵完全二叉树。</br>
将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。常见的堆有二叉堆、斐波那契堆等。</br>
Python自带的heapq模块实现的是最小堆，没有提供最大堆的实现。</br>
```
>>> import heapq
>>> dir(heapq)
['__about__', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_heapify_max', '_heappop_max', '_heapreplace_max', '_siftdown', '_siftdown_max', '_siftup', '_siftup_max', 'heapify', 'heappop', 'heappush', 'heappushpop', 'heapreplace', 'merge', 'nlargest', 'nsmallest']
```

### 创建堆
#### 直接由现有列表转为堆 heapify(list_name)
heapify(list_name) 可将列表以最少的以为操作转变为合法的小根堆（具有堆特征）。</br>
注意：heapify会直接将原列表修改为具有堆特征的列表！！</br>
```
>>> lst=[3,51,22,127,66,8,9,60,87,95]
>>> heapq.heapify(lst)
>>> lst
[3, 51, 8, 60, 66, 22, 9, 127, 87, 95]
>>> type(lst)
<class 'list'>
```
#### 通过迭代来创建堆 heappush(heap_name,x)
heappush(heap_name,x)：将x压入到堆heap_name中。可先建一个空列表，然后通过迭代一个可迭代对象并利用heappush来将这些元素逐个压入堆的方法来创建具有堆特征的列表。</br>
</br>
假设有一不知其元素内部顺序的可迭代对象（数字）：
```
# shuffle 作用在于随机打乱某序列中元素的排列顺序
>>> from random import shuffle
>>> data=list(range(12,30))
>>> data
[12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
>>> shuffle(data)
>>> data
[17, 19, 27, 12, 26, 22, 18, 28, 29, 15, 24, 23, 14, 21, 20, 25, 16, 13]
```
根据可迭代对象创建堆：
```
>>> heap=[]
>>> for n in data:
...     heapq.heappush(heap,n)
... 
>>> heap
[12, 13, 14, 15, 17, 18, 20, 19, 16, 26, 24, 27, 23, 22, 21, 28, 25, 29]
>>> 
```
压入新元素到已有堆中：
```
>>> lst
[3, 51, 8, 60, 66, 22, 9, 127, 87, 95]
>>> heapq.heappush(lst,32)
>>> lst
[3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66]
```

### 堆的基本操作
堆的基本操作只能给具有堆特征的列表使用，不能直接给列表使用。</br>
使用堆函数前需要确保列表已具备堆特征，可通过heappush或heapify来对列表进行处理。</br>

#### heappop 弹出最小值
heappop(heap_name)：由于python中默认创建的堆是小顶堆，所以最小值一定是列表对应index为0的位置，则每次弹出的都是heap_name[0] 位置上的元素。</br>
注意：heappop会直接修改原有列表。</br>
```
>>> heap
[12, 13, 14, 15, 17, 18, 20, 19, 16, 26, 24, 27, 23, 22, 21, 28, 25, 29]
>>> heapq.heappop(heap)
12
>>> heap
[13, 15, 14, 16, 17, 18, 20, 19, 29, 26, 24, 27, 23, 22, 21, 28, 25]
>>> heapq.heappop(heap)
13
>>> heap
[14, 15, 18, 16, 17, 23, 20, 19, 29, 26, 24, 27, 25, 22, 21, 28]
>>> heapq.heappop(heap)
14
>>> heap
[15, 16, 18, 19, 17, 23, 20, 28, 29, 26, 24, 27, 25, 22, 21]
```
#### heapreplace 弹出最小元素同时压入一个新元素
heapreplace(heap_name,x) ：弹出最小元素同时压入一个新元素，效率较高。</br>
```
>>> heap
[15, 16, 18, 19, 17, 23, 20, 28, 29, 26, 24, 27, 25, 22, 21]
>>> heapq.heapreplace(heap,25.6)
15
>>> heap
[16, 17, 18, 19, 24, 23, 20, 28, 29, 26, 25.6, 27, 25, 22, 21]
```

#### nlargest、nsmallest 找出n个最大或最小值
nlargest(n,iter) 和 nsmallest(n,iter) ：找出可迭代对象的n个最大值或最小值，效率比先普通排序再找更高。</br>
注意：这两个函数可以作用于普通列表，但是它们在heapq模块中。</br>
```
# 具有堆特征的列表
>>> lst
[3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66]
>>> heapq.nlargest(6,lst)
[127, 95, 87, 66, 60, 51]
>>> heapq.nsmallest(6,lst)
[3, 8, 9, 22, 32, 51]
# 没有堆特征的列表
>>> heapq.nsmallest(6,[2,74,73,85,3,5,9,12])
[2, 3, 5, 9, 12, 73]
```

## 双端队列（及其他集合）
双端队列deque ：</br>
双端队列是一种特殊的数据结构，特指允许两端都可以进行入队（插入）和出队（删除）操作的队列，其元素的逻辑结构仍是线性结构，具有队列和栈的性质。将队列的两端分别称为前端和后端，两端都可以入队和出队。</br>
输出受限的双端队列：允许在一端进行插入和删除，但在另一端只允许插入的双端队列。</br>
输入受限的双端队列：允许在一端进行插入和删除，但在另一端只允许删除的双端队列。</br>
</br>
Pyhon 中有双端队列 deque 和其他一些 collection 类型，它们被包含在模块 collections 当中。
```
>>> import collections
>>> dir(collections)
['AsyncGenerator', 'AsyncIterable', 'AsyncIterator', 'Awaitable', 'ByteString', 'Callable', 'ChainMap', 'Collection', 'Container', 'Coroutine', 'Counter', 'Generator', 'Hashable', 'ItemsView', 'Iterable', 'Iterator', 'KeysView', 'Mapping', 'MappingView', 'MutableMapping', 'MutableSequence', 'MutableSet', 'OrderedDict', 'Reversible', 'Sequence', 'Set', 'Sized', 'UserDict', 'UserList', 'UserString', 'ValuesView', '_Link', '_OrderedDictItemsView', '_OrderedDictKeysView', '_OrderedDictValuesView', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__path__', '__spec__', '_chain', '_collections_abc', '_count_elements', '_eq', '_heapq', '_iskeyword', '_itemgetter', '_nt_itemgetters', '_proxy', '_recursive_repr', '_repeat', '_starmap', '_sys', 'abc', 'defaultdict', 'deque', 'namedtuple']
>>> collections.__all__
['deque', 'defaultdict', 'namedtuple', 'UserDict', 'UserList', 'UserString', 'Counter', 'OrderedDict', 'ChainMap', 'Awaitable', 'Coroutine', 'AsyncIterable', 'AsyncIterator', 'AsyncGenerator', 'Hashable', 'Iterable', 'Iterator', 'Generator', 'Reversible', 'Sized', 'Container', 'Callable', 'Collection', 'Set', 'MutableSet', 'Mapping', 'MutableMapping', 'MappingView', 'KeysView', 'ItemsView', 'ValuesView', 'Sequence', 'MutableSequence', 'ByteString']
```

### 创建双端队列
双端队列可由其他可迭代对象创建，有多种创建方法，可用迭代方式创建，也可直接转换。</br>
#### 由其他类型转换成双端队列
deque(iter)：collections模块下的deque()函数，可将可迭代对象转变为deque类型。</br>
注意：python有独立的deque类型，deque()会新建一个deque类型对象，不会直接对原对象。</br>
```
>>> lst
[3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66]
>>> type(lst)
<class 'list'>
>>> collections.deque(lst)
deque([3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66])
>>> type(lst)
<class 'list'>
>>> type(collections.deque(lst))
<class 'collections.deque'>
```

### 双端队列的基本操作
双端队列和列表最大的区别是，双端队列可以快速地从左侧添加或删除元素，而列表不能。</br>
#### 添加元素
注意：append()和appendleft()均不属于collections模块下的函数，所以不需要添加模块名。</br>
```
# 在右侧添加元素
>>> deque_lst
deque([3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66])
>>> deque_lst.append('abc')
>>> deque_lst
deque([3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66, 'abc'])
# 在左侧添加元素
>>> deque_lst.appendleft('666')
>>> deque_lst
deque(['666', 3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66, 'abc'])
```

#### 弹出（删除）元素
```
# 在右侧弹出元素
>>> deque_lst.pop()
'abc'
>>> deque_lst
deque(['666', 3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66])
# 在左侧弹出元素
>>> deque_lst.popleft()
'666'
>>> deque_lst
deque([3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66])
```

#### 旋转元素
```
# 向右移n个单位，从右侧弹出的元素重新按弹出顺序插入到deque的左侧
>>> deque_lst
deque([3, 32, 8, 60, 51, 22, 9, 127, 87, 95, 66])
>>> deque_lst.rotate(1)
>>> deque_lst
deque([66, 3, 32, 8, 60, 51, 22, 9, 127, 87, 95])
>>> deque_lst.rotate(5)
>>> deque_lst
deque([22, 9, 127, 87, 95, 66, 3, 32, 8, 60, 51])
# 向左移n个单位，从右侧弹出的元素重新按弹出顺序插入到deque的右侧
>>> deque_lst
deque([22, 9, 127, 87, 95, 66, 3, 32, 8, 60, 51])
>>> deque_lst.rotate(-1)
>>> deque_lst
deque([9, 127, 87, 95, 66, 3, 32, 8, 60, 51, 22])
>>> deque_lst.rotate(-3)
>>> deque_lst
deque([95, 66, 3, 32, 8, 60, 51, 22, 9, 127, 87])
```
