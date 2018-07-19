# Python 面向对象：Special method 魔法方法、property特性、iterator 迭代器
## 魔法方法（特殊名称） Special method
魔法方法（特殊名称） Special method：形式为\_\_XXX\_\_的一些特殊名称。</br>
魔法方法的用途有很多，大多是为高级用途作准备。</br>
详情见Python官方文档 Special method names </br>
https://docs.python.org/3.7/reference/datamodel.html?highlight=__init__#object.__init__

### \_\_init\_\_ 构造函数constructor  
\_\_init\_\_ 构造函数constructor：</br>
构造函数\_\_init\_\_是Special method的中最常用的一种，通常在新建对象（类）时被调用，用于给对象定义其属性（初始化新建对象的状态）。</br>

#### 重写类的普通方法及其属性（构造函数）
当新建一个子类在超类之下时，子类会自动继承超类的所有方法，但是子类不会继承超类的属性。</br>
如果子类需要具备超类的属性，就需用调用未关联的超类构造函数或使用函数super()。</br>

```
class Bird:
    def __init__(self):		# 这个就是构造函数
        self.hungry=True
    def eat(self):
        if self.hungry == True:
            print('The bird ate something.')
            self.hungry=False
        else:
            print('No.Thank you!')
    def squawk(self):
        print('Squawk!Squawk!')



penguin=Bird()		# 实例化
penguin.eat()			# 调用方法
penguin.eat()
penguin.squawk()


# 输出结果
The bird ate something.
No.Thank you!
Squawk!Squawk!
```
```
# 创建一个在Bird下的子类
class SwimingBird(Bird):
    def __init__(self):
        self.swiming=['~'.join(list('Swiming'))]
    def swim(self):		# 重写普通方法
        print(self.swiming)

duck=SwimingBird()	# 创建实例
duck.swim()			# 调用当前类方法
duck.squawk()			# 调用超类方法


# 输出结果
['S~w~i~m~i~n~g']
Squawk!Squawk!

```

子类实例尝试调用含有超类属性的方法：
调用失败，AttributeError: 'SwimingBird' object has no attribute 'hungry'，即是子类 'SwimingBird'没有超类属性 'hungry'。

```
# 子类实例调用超类和超类属性有关的方法
duck.eat()

# 输出结果
Traceback (most recent call last):
  File "python3_u9_specialmethod.py", line 38, in <module>
    duck.eat()
  File "python3_u9_specialmethod.py", line 13, in eat
    if self.hungry == True:
AttributeError: 'SwimingBird' object has no attribute 'hungry'
```


#### 调用未关联的超类构造函数（超类属性）
在子类中的构造函数中，添加一行格式为“超类名.\_\_init\_\_(self)”的语句，即可让子类具备超类的属性。</br>
因为是在类里面进行调用构造函数，还没有实例化，所以称为“未关联”。</br>
当子类的实例调用方法时，python会将超类的self自动关联到该实例。</br>
这种方法多用于旧版python，新版本python建议用super()。</br>

```
class Bird:
    def __init__(self):
        self.hungry=True
    def eat(self):
        if self.hungry == True:
            print('The bird ate something.')
            self.hungry=False
        else:
            print('No.Thank you!')
    def squawk(self):
        print('Squawk!Squawk!')

class SwimingBird(Bird):
    def __init__(self):
        Bird.__init__(self)     # 调用超类未关联的构造函数
        self.swiming=['~'.join(list('Swiming'))]
    def swim(self):
        print(self.swiming)

duck=SwimingBird()
duck.swim()
duck.squawk()

duck.eat()

# 输出结果
['S~w~i~m~i~n~g']
Squawk!Squawk!
The bird ate something.
```

#### 使用函数super()
调用函数super()，可在子类的构造函数中，添加一行格式为“super().\_\_init\_\_()”的代码。</br>
在python3中调用super()，可不提供任何参数，一般也这么做。</br>
新版本python应该使用super()来给子类调用超类的属性，super()仅适用于新式类。</br>
如果当前类有多个超类，也只需添加一条“super().\_\_init\_\_()”的代码，就可以让当前类继承其所有超类的属性，而不像调用超类构造函数那样需要多条代码。</br>

```
# 修改上述代码
class Bird:
    def __init__(self):
        self.hungry=True
    def eat(self):
        if self.hungry == True:
            print('The bird ate something.')
            self.hungry=False
        else:
            print('No.Thank you!')
    def squawk(self):
        print('Squawk!Squawk!')

class SwimingBird(Bird):
    def __init__(self):
        super().__init__()			# 调用函数super()让子类继承超类的属性
        self.swiming=['~'.join(list('Swiming'))]
    def swim(self):
        print(self.swiming)

duck=SwimingBird()
duck.swim()
duck.squawk()

duck.eat()
             
# 输出结果
['S~w~i~m~i~n~g']
Squawk!Squawk!
The bird ate something.
```

## Emulating container types
Python中可以用一些特殊方法来创建类似Python内置的序列和字典那样的自定义对象(Emulating container types)，但需要遵循Python关于序列和映射的一些协议。
详细见官方文档 3.3.7. Emulating container types 一节</br> 
https://docs.python.org/3.7/reference/datamodel.html?highlight=__init__#object.__iter__

### 映射和序列基本协议
#### 一些额外要求</br>
序列：</br>
key必须为0至n-1整数，可以为负整数，但x[-n]必须和x[len(x)-n]等价；</br>
key的类型必须为整数，不能是其他类型，如果不符合这个要求，则应引发TypeError；</br>
key如果超出了要求的0到n-1的范围，应引发IndexError。</br>

字典：</br>
key没有类型要求;</br>
如果查找时所查找的key不存在于该字典中，则应引发KeyError。</br>

#### 4个特殊方法
不可变对象适用前两个方法，可变对象适用以下4个方法。</br>

* \_\_len(self)\_\_ 返回结合包括的项数</br>
\_\_len(self)\_\_ ：特殊方法，返回结合包括的项数。</br>
参数：self（调用时self是不用打的，也即是调用时不需要参数，但新建类时需要）</br>
返回值：序列返回元素个数、字典返回key-value对数</br>
注意：</br>
如果\_\_len(self)\_\_的返回值是0，则会被布尔上下文视为假；</br>
如果类没有定义\_\_len(self)\_\_这个方法，则默认这个类下的实例可以无限长。</br>

* \_\_get\_\_(self,key) 查找key对应的value</br>
\_\_getitem\_\_(self,key)：特殊方法，根据key查找对象中key对应的值。</br>
参数：self、key</br>
返回值：value</br>

* \_\_setitem\_\_(self,key,value) 给key设置对应的value</br>
\_\_setitem\_\_(self,key,value)：特殊方法，给key设置对应的value，适用于可以变对象。</br>
参数：self、key、value</br>
返回值：无</br>

* \_\_delitem\_\_(self,key) 删除对象中与key关联的值</br>
\_\_delitem\_\_(self,key)：特殊方法，删除对象中与key关联的值，适用于可变对象。</br>
参数：self、key</br>
返回值：无</br>
注意：如果类中没有实现\_\_delitem\_\_(self,key)，则表示禁止删除这个类中实例的元素。</br>

```
def checkIndex(key):
    if not isinstance(key,int):
        # 其实是想规定输入的key必须是数字才设TypeError，一般字典用KeyError
        raise TypeError
    if key < 0:
        # 禁止index小于0，小于0一般用于查询，但因此处有新建元素，所以要求大于0
        raise IndexError

class ArithmeticSequence:
    def __init__(self,start=0,step=1): # 新建子类，初始值为0，步长为1
        self.start=start
        self.step=step
        self.change={}

    def __getitem__(self,key):
        checkIndex(key)   # 检查key是否合法             
        try:
            return self.change[key]
        except KeyError:  # 如果key不存在于该字典中则引发keyError并执行以下操作
            return self.start+key*self.step  # 计算元素的值

    def __setitem__(self,key,value):
        checkIndex(key)
        self.change[key]=value

s1=ArithmeticSequence()
print(s1.change)

# 查看符合规则的序列的值
print(s1.__getitem__(0))
print(s1.__getitem__(9))

# 按照用户意愿修改的序列的值
s1.__setitem__(4,6)
print(s1.__getitem__(4))
print(s1.change)

# 修改序列的步长和初始值
s2=ArithmeticSequence(3,4)
print(s2.__getitem__(5))
print(s2.__getitem__(105))
s2.__setitem__(233,666)
print(s2.__getitem__(233))
print(s2.change)

# 尝试输入不合法key
s2.__setitem__('hello',4)



# 输出结果
{}
0
9
6
{4: 6}
23
423
666
{233: 666}
Traceback (most recent call last):
  File "python3_u9_specialmethod_emulatingcontainertypes.py", line 54, in <module>
    s2.__setitem__('hello',4)
  File "python3_u9_specialmethod_emulatingcontainertypes.py", line 30, in __setitem__
    checkIndex(key)
  File "python3_u9_specialmethod_emulatingcontainertypes.py", line 11, in checkIndex
    raise TypeError   
TypeError
```

### 作为Python内置序列或映射的子类
直接让新建类成为Python内置序列或映射的子类，可以让新建类继承list、dict、str等的内置方法，而不用每次自己重新定义，只需要使用函数super()继承超类属性并写一下子类自己需要的方法即可。</br>
字典：</br>
推荐提供keys(), values(), items(), get(), clear(), setdefault(), pop(), popitem(), copy(), update()的方法。</br>
可变序列：</br>
应提供append(), count(), index(), extend(), insert(), pop(), remove(), reverse(), sort()的方法。</br>
所有序列：</br>
应提供加号表示连接、乘号表示重复的方法，可通过\_\_add\_\_(), \_\_radd\_\_(), \_\_iadd\_\_(), \_\_mul\_\_(), \_\_rmul\_\_() and \_\_imul\_\_() 等途径实现。</br>
字典和序列：</br>
推荐使用\_\_contains\_\_()来实现对in的有效使用，字典根据key来搜索，序列根据value来搜索；</br>
推荐使用 \_\_iter\_\_()来实现对字典或序列的迭代，字典根据key来迭代，序列根据value来迭代。</br>

```
# 错误例子
>>> class Counterlist(list):
...     def __init__(self,*args):
...         super().__init__()	# 出错在于这句
...         self.counter=0
...     def __getitem__(self,index):
...         self.counter+=1
...         return super(Counterlist,self).__getitem__(index)
... 
>>> cl=Counterlist(range(10))
>>> cl						# 实例是空列表
[]		
```
```
# 正确例子
>>> class Counterlist(list):
...      def __init__(self,*args):
...          super().__init__(*args)		# 修改了参数
...          self.counter=0
...      def __getitem__(self,index):		
...          self.counter+=1			# 查询一次计算一次查询次数
...          return super(Counterlist,self).__getitem__(index)
... 
>>> cl=Counterlist(range(10))
>>> cl								# 实例非空
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> cl.reverse()
>>> cl
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
>>> del cl[6:8]
>>> cl
[9, 8, 7, 6, 5, 4, 1, 0]
>>> cl.__getitem__(3)
6
>>> cl.counter
1
>>> cl.__getitem__(7)
0
>>> cl.counter
2
>>> cl[4]
5
>>> cl.counter
3
>>> cl[5]+cl[2]
11
>>> cl.counter		# 每访问一次列表元素，self.counter自动加1
5
```

## property 特性
Python中可以通过两条下划线设置某属性私有，这样可以对访问和修改进行限制。在设为私有的前提下，如果还想访问或修改属性的内容，则应该建立对应的方法，如get_name、set_name等，同时在方法中指定属性修改的条件。但是，当类的属性很多的时候，实例化时给每个属性都逐个分别调用get和set的方法并不现实，所以Python提供了解决这问题的一个方案。使用函数property后，可以通过普通调用方法来访问及修改私有属性，而不需要去专门调用该属性的方法。</br>
Python中提供的通过存取方法定义的属性，称为“特性 property”。</br>
Python中有两种创建特性的机制，较新的方法是使用函数property，但它仅能用于新类。</br>

Python官方文档</br>
Descriptor HowTo Guide中Properties一节</br>
https://docs.python.org/3.7/howto/descriptor.html?highlight=property </br>
Built-in Functions中class property(fget=None, fset=None, fdel=None, doc=None) 一节</br>
https://docs.python.org/3.7/library/functions.html#property </br>

### 函数 property
property(fget,fset,fdel,doc)：参数是访问、修改、删除属性的方法名。前两个参数为必须，后两个备选。参数的顺序是固定的，获取在前，设置在后。</br>
参数：</br>
没有指定任何参数，则特性不可读也不可写；仅指定fget则特性read-only。</br>
fget - getting an attribute value</br>
fset - setting an attribute value</br>
fdel - deleting an attribute value</br>
doc - creates a docstring for the attribute</br>


[对比面向对象-访问限制里的例子写法](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%EF%BC%9A%E7%B1%BB%E3%80%81%E5%AE%9E%E4%BE%8B%E3%80%81%E5%B1%9E%E6%80%A7%E3%80%81%E6%96%B9%E6%B3%95.MD#%E8%AE%BF%E9%97%AE%E9%99%90%E5%88%B6)
```
class Commodity:
    def __init__(self,name,price):
        self.__name=name
        self.__price=price

    def get_name(self):
        return self.__name              # 查询产品名称

    def get_price(self):
        return self.__price             # 查询产品价格

    def change_price(self,price):
        if price < 0 :
            print('The price cannot be smaller than 0.') # 输入不正确时拒绝调整 
        else:
            self.__price=price  # 更新价格
def sell(self):
        print(self.__name.title(),'is on sell.')

    name=property(get_name)     # read-only
    price=property(get_price,change_price,'The product price!!')


commodity1=Commodity('milk',4)
commodity2=Commodity('cake',60)
commodity3=Commodity('egg',2)

print(commodity1.name,commodity1.price)
print(commodity2.price)
commodity2.price=163			# 直接对价格进行修改
print(commodity2.price)
print('The price of {} is {}.'.format(commodity3.name,commodity3.price))


# 输出结果：
milk 4
60
163
The price of egg is 2.
```

### 装饰器decorator
Python内置的装饰器@property可以实现和函数property()相同的功能，但语法上不一样。</br>

用装饰器改写上面的程序：</br>
注意访问和修改的方法名都变为了属性名！！</br>
```
class Commodity:
    def __init__(self,name,price):
        self.__name=name
        self.__price=price

    @property
    def name(self):
        return self.__name              # 查询产品名称

    @property
    def price(self):
        return self.__price             # 查询产品价格

    @price.setter
    def price(self,value):
        if value < 0 :
            print('The price cannot be smaller than 0.') # 输入不正确时拒绝调整 
        else:
            self.__price=value  # 更新价格

    def sell(self):
        print(self.__name.title(),'is on sell.')


commodity1=Commodity('milk',4)
commodity2=Commodity('cake',60)
commodity3=Commodity('egg',2)

print(commodity1.name,commodity1.price)
print(commodity2.price)
commodity2.price=163
print(commodity2.price)
print('The price of {} is {}.'.format(commodity3.name,commodity3.price))




# 输出结果
milk 4
60
163
The price of egg is 2.
```

### 用于访问对象属性的special method
property不支持旧类，旧类中使用某些魔法方法来实现新类中近似property的功能。</br>
但是，使用魔法方法来实现属性的访问和修改时，需要额外的细节处理，所以可以用property的情况下，最好优先选择property。</br>

官方文档见 3.3.2. Customizing attribute access 一节</br>
https://docs.python.org/3.7/reference/datamodel.html?highlight=__getattr__#object.__getattr__ </br>

* object.\_\_getattr\_\_(self, name)</br>
object.\_\_getattr\_\_(self, name)：return the (computed) attribute value or raise an AttributeError exception</br>
参数：实例和属性名</br>
返回：属性存在时返回属性值，不存在时raise an AttributeError exception（需要人为去写）。</br>
注意：如果按照正常的机制（如属性并没有设置私有，不需要额外使用方法访问时）就可以找到属性，则不会调用object.\_\_getattr\_\_(self, name)，这样做目的在于提高效率。</br>

* object.\_\_getattribute\_\_(self, name)</br>
object.\_\_getattribute\_\_(self, name)： return the (computed) attribute value or raise an AttributeError exception</br>
参数：实例和属性名</br>
返回：属性存在时返回属性值，不存在时raise an AttributeError exception（需要人为去写）。</br>
注意：仅有新类可以用此方法，当同时有object.\_\_getattr\_\_(self, name)，则优先使用object.\_\_getattr\_\_(self, name)。\_\_getattribute\_\_有可能导致无限循环，所以它的实现应该始终调用具有相同名称的基类方法来访问它所需要的任何属性。</br>

* object.\_\_setattr\_\_(self, name, value)</br>
object.\_\_setattr\_\_(self, name, value)：属性赋值</br>
参数：实例、属性名、属性的新值</br>
返回：无</br>
注意：object.\_\_setattr\_\_(self, name, value)也有可能导致无限循环，所以它也需要始终调用同名的基类方法。</br>


* object.\_\_delattr\_\_(self, name)</br>
object.\_\_delattr\_\_(self, name)：attribute deletion</br>
参数：实例和属性名</br>
返回：无</br>
注意：实例必须含有需要删除的那个属性</br>

## 迭代器 Iterator
iterate 迭代</br>
迭代器：通过给用户定义的对象（如用户创建的类及其实例）添加\_\_iter\_\_方法，让其成为可迭代的对象，再通过循环for ... in ... 来实现该对象的迭代。 iterator objects必须遵循迭代协议。</br>
在一般情况下，使用迭代器比使用序列或字典迭代更通用、简单、节省内存。</br>
官方文档见 https://docs.python.org/3.7/library/stdtypes.html#iterator-types</br>

### 迭代器协议 iterator protocol
* iterator.\_\_iter\_\_()</br>
iterator.\_\_iter\_\_()：Return the iterator object itself.</br>
参数：无</br>
返回值：迭代器本身</br>

* iterator.\_\_next\_\_()</br>
iterator.\_\_iter\_\_()：Return the next item from the container. If there are no further items, raise the StopIteration exception. </br>
参数：无</br>
返回值：迭代器的下一个值，如果没有则引起StopIteration exception。</br>
注意：For Python 2.x, you should change \_\_next\_\_() to next().</br>

```
class FibsNumber:
    def __init__(self):
        self.st=0
        self.nd=1

    def next(self):   # 尽管用的python3，但用__next__()还是会报instance has no next() method
        try:
            self.st,self.nd=self.nd,self.st+self.nd
            return self.st
        except StopIteration:
            raise StopIteration

    def __iter__(self):
        return self

fibs=FibsNumber()

for num in fibs:
    print(num)
    if num > 999:
        break


# 输出结果：
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
```

### 从迭代器创建序列
可使用list(instance)将迭代器转换为列表（其他的构造函数转换为对应的序列），但需要在next()中定义结束条件，不然会一直迭代下去。
```
class FibsNumber:
    def __init__(self):
        self.st=0
        self.nd=1

    def next(self):
        self.st,self.nd=self.nd,self.st+self.nd
        if self.st > 999:
            raise StopIteration
        return self.st

    def __iter__(self):
        return self

fibs=FibsNumber()

print(list(fibs))

# 输出结果
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987]
```

## 生成器 generator
生成器：包含关键字yield的用普通函数来定义的迭代器，一般情况下可以不使用生成器。</br>
生成器由两个部分组成：</br>
生成器的函数 - 由def定义，其中包含yield语句；</br>
生成器的迭代器 - def定义的函数的返回结果，可以像使用其他迭代器那样使用它们。</br>
注意：生成器不能用于字符窜和类似于字符串的对象的迭代</br>

```
def flatten(nested):
    try:
        for sublist in nested:
            for element in flatten(sublist):
                yield element
    except TypeError:
        yield nested

student_ID=[2,4,[454,777],[7,4545],[9878,7]]
print(list(flatten(student_ID)))

for num in flatten(student_ID):
    print(num)
                    
                              
# 输出结果
[2, 4, 454, 777, 7, 4545, 9878, 7]
2
4
454
777
7
4545
9878
7
```
