# Python 面向对象
Python是一种面向对象的语言。</br>
对象：一系列数据（属性），以及一套访问和操作这些数据的方法。</br>
面向对象的好处：</br>
1)多态：可对不同类型的对象进行相同操作；</br>
2)封装：对外部隐藏有关对象的原理细节；</br>
3)继承：可基于通用类创建出专用类。</br>

## 抽象原则的基本定义
### 多态
多态 polymorphism：尽管不知道变量指向的是哪种对象，也能对其执行操作调用其方法。</br>
多态是Python编程核心，有时被称作“Duck_typing”译文“鸭子类型”。我个人觉得这个duck应该是“隐蔽、躲在下面”的意思，对应词组“duck into something”，也就是多态的作用的意译。</br>

按照之前的方法，必须先知道变量指向的是什么类型的数据，才能执行对应的操作或方法。</br>
例如，变量指向的是字符串，则需要使用字符串的方法来对其进行处理，变量指向的是列表，则需要用列表方法。</br>

如果使用对象，就不需要在意这种问题，不需要对变量执行检查就可以执行操作或方法，这被称之为“多态”。</br>
实际上，很多Python的内置运算符和函数都是多态的，支持对不同类型序列的处理。</br>

```
import random	# 模块random
x=random.choice(['hello world','Sharon',[1,3,6,'e'],('a','c',3)])	# choice()随机选择一个序列抛出，抛出结果类型不确定
print(x)

print(x.count('o'))	# 对象方法，对抛出结果进行字母o出现次数的计数
                     
# 每次抛出结果都不一样，但无论x是什么类型，都可以执行object.count()方法
$ python3 python_u7_object.py
[1, 3, 6, 'e']
0
$ python3 python_u7_object.py
hello world
2
$ python3 python_u7_object.py
('a', 'c', 3)
0
$ python3 python_u7_object.py
hello world
2
```

### 封装
封装：</br>
面向对象编程的一个重要特点就是数据封装。每个类中的实例基本都包含了实现功能所需要的数据（属性），那么就没必要在外部写函数，可以直接在类的定义中来处理这些数据。</br>
此处的程序被称之为“类的方法”，这种处理方式被称为“封装”。



### 继承
继承：</br>
当创建一个新类时，需要定义其方法。如发现已有类的方法与需要定义的新类的方法功能类似，那么可以让新类在调用自己的方法时，直接调用已有类的方法，而不用重复写代码。</br>


## 类 class
### 类的定义
类 class ：由方法定义，即根据其特征（属性）考虑它能实现些什么功能（对象的方法）来作出的分类。</br>
类：这是什么？（名词） </br>
属性：它有什么特征？（形容词 attribute） </br>
方法：它能被用来干嘛？（动词）</br>

### 类与类间的关系
实例 Instance：某只雀（实例对象） </br>
子类：麻雀（类对象）</br>
超类：鸟（类对象）</br>
实例是根据类创建出来的一个个具体的“对象”，同类的每个对象都拥有相同的方法，但各自的数据可能不同。</br>

### 类的属性
Python的属性查找规则：至下向上，先查实例属性，找不到实例属性时，再查类属性。</br>

### 实例、类的方法
超类方法 ∈ 子类方法 ∈ 实例方法</br>
方法数量：实例>子类>超类，实例中包含的但不需要的超类方法需要被设置为“什么都不做”或引起异常。</br>

### 类 class与类型type的关系
Python3不再严格区分类class和类型type。</br>
Python2及其之前版本有分，要在定义类前添加以下一行语句，Python3不用。</br>
```
__metaclass__ = type
```

个人认为：</br>
class 是按特征分的类，特征决定其功能（对象的方法）。类的分类方法不固定，可以自定义。类和类之间可能有比较多近似的特征，但是同一类中的实例之间也可能有比较多的不同点。只是因为它们的某种特征相似而被分在一起而已。例如，英语好和英语不好，编程好和编程不好，男的和女的，此时英语好编程好的女孩子是可以根据情况的不同来被分类到不同的类中的。</br>
type 是被设计者在Python里面写死的，它们的分类方法是固定的。类型和类型之间有比较多的不同，相同类型的数据会有比较多相同的特征。</br>
逻辑上，type也可以是一种class，是class的其中一个分支。</br>

## 创建自定义类过程
定义类：class语句后接类名，类名为大写字母开头，格式是有点像函数定义的格式，需缩进，内容是类的方法；</br>
创建实例：实例格式“variable=类名()”，“类名()”为variable是指向这个实例的变量名；</br>
给实例绑定属性：“实例名.属性名=具体属性内容”。</br>

### 定义类

```
# class定义类，类名为Student，pass的位置其实应填类的方法
>>> class Student:
...     pass
... 
>>> 
```

### 创建实例
```
# 创建实例，Student()为一个实例对象，此处建立了3个对象，括号左边为指向实例的变量名（实例名）
>>> student1=Student()
>>> student2=Student()
>>> student3=Student()
```

### 属性
#### 给实例绑定属性
```
# 给实例对象绑定一个叫name和number的属性
>>> student1.name='Sharon'		# 绑定属性
>>> student2.name='Lulu'
>>> student3.name='Ben'
>>> student2.name				# 查看属性
'Lulu'
>>> student3.name
'Ben'
>>> student1.number=201132000
>>> student1
<__main__.Student object at 0xb703c9ec>
>>> student1.number
201132000
```

若要在创建实例前，给所有实例绑定一个固定的属性，可在class里进行定义。
实例self代表实例整体，右边指的是属性，在创建实例时输入参数会得到更新。
可把这种定义属性的方式看作一种方法的定义，参数为对象名和属性名，括号内第一个参数一定为self。


#### 常规写法：定义类同时绑定类的属性
```
# 给类Student绑定gender性别和grade年级的属性，__init__ 特殊方法禁止参数为空。
>>> class Student:
...     def __init__(self,gender,grade):
...         self.grade=grade
...         self.gender=gender
... 

# 创建一个新的实例并输入其属性
>>> student4=Student('girl',1)
>>> student4.grade
1

# 这种定义方法会影响这个类下已存在的实例，其实相当于创建了一个和原来的Student无关的新类
>>> student2=Student('girl',3)	# 重新以Student()的固定属性定义student2
>>> student2.gender
'girl'
>>> student2.name				# student2原有的name属性没有了
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'name'
>>> student3.name				# student3 原有的name属性还在
'Ben'
>>> 
```

### 方法
#### 常规写法
```
# 给类新建方法，参数为self，注意show_gender那一行
>>> class Student:
...     def __init__(self,gender,grade):
...         self.gender=gender
...         self.grade=grade
...     def show_gender(self):
...         print('The {} gender is {}'.format(self,self.gender))
... 
# 每一次使用class都相当于定义了一个新类了，原来属于这个类的实例已经不属于这个类了
>>> student4.show_gender()		
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute 'show_gender
>>> student5=Student('boy',6)
# 实例对象方法调用
>>> student5.grade
6
>>> student5.show_gender()
The <__main__.Student object at 0xb703c9ac> gender is boy
```

#### 以外部函数的形式给类写方法
可以在类定义之外建立一个普通函数，将其关联到类的方法。</br>
(我个人觉得这种做法有点违反封装原则... )</br>
注意：需要关注外部函数对self的调用情况。</br>

```
class Student:
    def __init__(self,name):
        self.name=name

    def say(self):
        print('Hello')  # 可以改为pass，需要的时候再将这个方法关联到外部函数

# 在类自定义外部的函数，没有带参数self
def new_way():
    print('Hi!')

student1=Student('Sharon')
student2=Student('Ben')

student2.say()
student2.say=new_way  	# 将方法关联到外部函数
student2.say()				# 执行方法

# 输出结果
Hello
Hi!						# 关联到外部函数的方法的执行结果
```

尝试在外部函数调用self参数
```
class Student:
    def __init__(self,name):
        self.name=name

    def say(self):
        print('Hello')  # 可以改为pass，需要的时候再将这个方法关联到外部函数

# 新增加的外部函数，函数内部需要调用self
def new_way2():
print('Hello',self.name)

student1=Student('Sharon')
student2=Student('Ben')

student2.say()=new_way2
student2.say()

# 运行结果：会出错，这种写法无法调用self，所以想调用self的方法要写在类定义的内部
$ python3 python_u7_object.py
  File "python_u7_object.py", line 50
    student2.say()=new_way2
    ^
SyntaxError: can't assign to function call
```
要想实现在类定义外的函数可以调用self，可以改为以下这种写法:
```
class Student:
    def __init__(self,name):
        self.name=name

    def say(self):
        new_way2(self)  # 调用外部函数，并将参数self传给外部函数

def new_way2(a):
print('Hello',a.name)

student1=Student('Sharon')
student2=Student('Ben')

student2.say()

# 输出结果
Hello Ben
```

#### 给方法起名字
```
class Student:
    def __init__(self,name,number,gender,classnum,grade):
        self.name=name
        self.number=number
        self.gender=gender
        self.classnum=classnum
        self.grade=grade

    def info(self):
        print('The student {} informations'.format(self.name))
        print('gender:',self.gender)
        print('number:',self.number)
        print('class number:',self.classnum)
        print('grade:',self.grade)

student1=Student('Sharon',201132000,'girl',2,4)
student2=Student('Ben',201432065,'boy',5,1)

student1_list=student1.info		# 给方法起名字
student1_list()					# 用名字调用方法

The student Sharon informations
gender: girl
number: 201132000
class number: 2
grade: 4
```

### 类及其属性、方法、实例的创建完整流程
这个也是日常使用的定义方法：</br>
定义类 - 定义类属性 - 定义类方法 - 创建实例 - 输入实例的属性。</br>
以下程序为创建自定义类的完整过程，脚本模式下执行：</br>
```
# 创建类Student
class Student:
# 定义类的属性
    def __init__(self,name,number,gender,classnum,grade):
        self.name=name
        self.number=number
        self.gender=gender
        self.classnum=classnum
        self.grade=grade
# 定义类的方法
    def info(self):
        print('The student {} informations'.format(self.name))
        print('gender:',self.gender)
        print('number:',self.number)
        print('class number:',self.classnum)
        print('grade:',self.grade)

# 创建实例，绑定实例属性
student1=Student('Sharon',201132000,'girl',2,4)
student2=Student('Ben',201432065,'boy',5,1)

# 查询实例及其属性
print(student1)
print(student1.name)

# 调用实例方法
student2.info()



# 输出结果
<__main__.Student object at 0xb710248c>
Sharon
The student Ben informations
gender: boy
number: 201432065
class number: 5
grade: 1
```

## 访问限制
在Class内部，可以有属性和方法，而外部代码可以通过直接调用实例变量的方法来操作数据，这样，就隐藏了内部的复杂逻辑。然而，外部函数还是可以自由访问和修改实例的属性。</br>
如果想禁止外部代码访问和修改实例的属性，可在属性名前面加两个下划线__将其设为私有，禁止外部访问和修改属性，但不影响调用。</br>
在Python中，某种程度上说，一个下划线和两个下划线代表不同程度的私有度。</br>
（其实这种做法仅仅是种警告，要访问和修改还是有办法的，只是让人知道程序的编写者不想让别人访问和修改实例的属性，是一种约定。）

### 禁止访问
```
class Commodity:
    def __init__(self,name):
        self.name=name

    def sell(self):
        print(self.name.title(),'is on sell.')

commodity1=Commodity('milk')		# 创建实例并输入属性
commodity2=Commodity('cake')

commodity2.sell()
print(commodity1.name)			# 此处实例属性时可以被访问的

# 输出结果
Cake is on sell.		
milk
```

在属性前面添加两条下划线__将属性设为私有。
此处报错了，因为实例的属性是不能被外部访问的。

```
class Commodity:
    def __init__(self,name):
        self.__name=name

    def sell(self):
        print(self.__name.title(),'is on sell.')

commodity1=Commodity('milk')	
commodity2=Commodity('cake')

commodity2.sell()				# 此句没有出错，实例属性是可以正常被调用的
print(commodity1.__name)		# 出错的是这句，__ 禁止访问属性，删除此句程序会运行正常

# 输出结果
Traceback (most recent call last):
  File "python_u7_object.py", line 65, in <module>
    print(commodity1.__name)
AttributeError: 'Commodity' object has no attribute '__name'
```

### 禁止修改
双下划线不但禁止了实例属性的访问，还禁止了属性被外部修改。
```
class Commodity:
    def __init__(self,name):
        self.name=name			# 属性没有被设为私有

    def sell(self):
        print(self.name.title(),'is on sell.')

commodity1=Commodity('milk')
commodity2=Commodity('cake')		# 创建实例并输入其属性

commodity2.sell()

commodity2.name='cookies'			# 此句从外部修改了commodity2的name属性
commodity2.sell()


# 输出结果
Cake is on sell.
Cookies is on sell.					# commodity2的name属性被修改了
```




```
class Commodity:
    def __init__(self,name):
        self.__name=name					# 将属性设为私有

    def sell(self):
        print(self.__name.title(),'is on sell.')

commodity1=Commodity('milk')
commodity2=Commodity('cake')				# 创建实例及其属性

commodity2.sell()

commodity2.__name='cookies'				# 从外部修改实例commodity2的属性name
commodity2.sell()

# 输出结果
Cake is on sell.
Cake is on sell.								# commodity2的属性name没有被修改，且没有出现错误报告
```

### 适当访问和修改实例属性
如果真的需要在外部访问和修改实例属性，可以给类定义方法并限制条件来让外部在满足条件下修改属性。
```
class Commodity:
    def __init__(self,name,price):
        self.__name=name
        self.__price=price

    def get_name(self):
        return self.__name		# 查询产品名称

    def get_price(self):
        return self.__price		# 查询产品价格

    def change_price(self):
        new_price=int(input('Place input the new price:'))	# 获取新价格
        if new_price < 0 :
            print('The price cannot be smaller than 0.')		# 输入不正确时拒绝调整	
        else:
            self.__price=new_price						# 更新价格

    def sell(self):
        print(self.__name.title(),'is on sell.')


commodity1=Commodity('milk',4)
commodity2=Commodity('cake',60)
commodity3=Commodity('egg',2)

print(commodity1.get_price())		# 查询商品1的价格
print(commodity2.get_name())		# 查询商品2的名称

print(commodity3.get_price())		# 查询商品3价格
commodity3.change_price()			# 修改商品3价格
print(commodity3.get_price())		# 再次查询商品3价格


# 输出结果
4
cake
2
Place input the new price:3
3

# 再次执行脚本，最后三行的输出结果
2
Place input the new price:-9			# 不符合价格要求
The price cannot be smaller than 0.
2						# 商品3价格没有被修改
```


## 类的命名空间
类的命名空间：在类定义中的变量，可以被所有实例调用。
```
class Animal:
    zoo='CIZ'		# 变量zoo可以被Animal下的所有子类和实例调用
    def __init__(self,animalnum):
        self.animalnum=animalnum
    def showls(self):
        print(self.animalnum)
        print(self.zoo)

cat=Animal('0001')
cat.showls()

dog=Animal('0023')
dog.showls()

# 输出结果
0001
CIZ
0023
CIZ
```

## 指定超类
子类是通常超类（通用类）创建出来的专用类，子类可直接调用其超类的方法而不需要再重复定义，只需要定义子类的属性即可，即继承。</br>
格式：子类名写在前，超类名用圆括号括起。</br>
```
class 子类名（超类名）：
```
实例：过滤器
```
>>> class Filter:
...     def __init__(self,seq,blocked):
...         self.seq=seq			# 原序列
...         self.blocked=blocked		# 要过滤的成分
...     def filter(self):
...         return [x for x in self.seq if x not in self.blocked]	# 列表推导，执行过滤
... 
>>> ls1=Filter([1,2,3],[3])
>>> ls1.seq
[1, 2, 3]
>>> ls1.blocked
[3]
>>> ls1.filter()
[1, 2]
>>> 
>>> ls2=Filter(['abc','Sharon',3,4],['abc'])
>>> ls2.filter()
['Sharon', 3, 4]
>>> 
```
上面这个程序，实例的属性是包含序列本身和要过滤的成分的，如果想实例的属性只包含过滤的成分，可以改为以下方式。此程序中，实例的属性只包含要过滤的成分，而需要被过滤的序列会作为方法的参数传入到方法filter()中。
```
class Filter:
    def __init__(self,blocked):
        self.blocked =blocked		# 需要被过滤掉的成分
    def filter(self,sequence):
        return [x for x in sequence if x not in self.blocked]

f1=Filter(['abc'])					# 传入要被过滤掉的成分
print(f1.filter(['abc','Sharon',3,4]))		# 传入需要被过滤的序列，执行过滤后回传


# 输出结果
['Sharon', 3, 4]
```
当然也可以把过滤的成分写为固定的，这样就可以将Filter用作其他类的基类。这用就不需要重新写过滤的方法。
```
class Filter:
    def __init__(self):
        self.blocked =[]		# 不作任何过虑
    def filter(self,sequence):
        return [x for x in sequence if x not in self.blocked]

class SpaceFilter(Filter):			# 将SpaceFilter设置为Filter的子类
    def __init__(self):
        self.blocked=['']

f1=Filter()					# Filter的实例
print(f1.filter(['abc','Sharon',3,4,'']))
f2=SpaceFilter()				# SpaceFilter的实例
print(f2.filter(['abc','Sharon',3,4,'']))



# 输出结果
['abc', 'Sharon', 3, 4, '']
['abc', 'Sharon', 3, 4]
```


## 继承
继承：</br>
由一个已有类中，创建一个子类，子类可以直接调用原有类的方法，这种方式被称之为“继承”。</br>
过滤器的例子中，Filter是SpaceFilter的基类、父类、超类（Base class、Super class），SpaceFilter是Filter的子类（Subclass）。</br>
多重继承：</br>
同时继承多个类，格式为 class_Subclass(Base class1,Base_class2,...)，应避免过多的继承。如果基类中存在同名方法，Base class1的方法会覆盖Base_class2的方法，所以要注意基类的顺序（MRO）。
### issubclass(class1,class2) 子类判断
issubclass(class1,class2) ：判断class1是否为class2的子类，返回布尔型。（不能用于实例的检验）</br>
参数：要被判断的类</br>
返回值：布尔型</br>
```
class Filter:
    def __init__(self):
        self.blocked =[]		# 不作任何过虑
    def filter(self,sequence):
        return [x for x in sequence if x not in self.blocked]

class SpaceFilter(Filter):			# 将SpaceFilter设置为Filter的子类
    def __init__(self):
        self.blocked=['']

print(issubclass(SpaceFilter,Filter))
print(issubclass(Filter,SpaceFilter))

# 输出结果
True
False
```

### isinstance(instance,class) 实例判断
isinstance(instance,class) ：判断instance是否是class的实例。</br>
参数：实例名，类名</br>
返回值：布尔型</br>
注意：子类的实例也是超类的实例。</br>
```
class Filter:
    def __init__(self):
        self.blocked =[]
    def filter(self,sequence):
        return [x for x in sequence if x not in self.blocked]

class SpaceFilter(Filter):
    def __init__(self):
        self.blocked=['']

f1=Filter()
print(f1.filter(['abc','Sharon',3,4,'']))
f2=SpaceFilter()			# f2在SpaceFilter下实例化
print(f2.filter(['abc','Sharon',3,4,'']))

print(isinstance(f1,Filter))
print(isinstance(f2,Filter))	# f2也是Filter的实例

# 输出结果
True
True
```

### 获悉实例所属类
* instance.\_\_class\_\_
```
# 续上例
print(f1.__class__)
print(f2.__class__)
print(SpaceFilter.__class__)


# 输出结果
<class '__main__.Filter'>
<class '__main__.SpaceFilter'>
<class 'type'>
```

* type(instance)
```
# 续过滤器例子
print(type(f1))
print(type(f2))
print(type(SpaceFilter))

# 输出结果
<class '__main__.Filter'>
<class '__main__.SpaceFilter'>
<class 'type'>
```

### 查询、获取、设置对象属性
#### 查询、获取、设置对象单个属性
* hasattr(obj,attr)：has attribute 查询对象obj的属性中，是否有属性值attr存在，也可用于检验方法，返回布尔型。</br>
* getattr(obj,attr)：get attribute获取对象obj的属性attr，返回属性值。</br>
* setattr(obj,attr1,19)：set attribute 把对象obj的属性attr1设置为属性值19（19仅是例子），返回新的属性值。</br>
注意：要先证明存在再获取，如果获取不存在的属性会发生错误报告，也可改为getattr(obj,attr,None)不报错返回None。setattr()不能更新已存在的属性，只能新增一个全新的属性。
```
# 续过滤器例子
print(hasattr(f1,'blocked'))	# 对象f1属性blocked是否存在
print(hasattr(f2,'blocked'))
print(getattr(f2,'blocked'))	# 获得对象f2的属性blocked的值
print(f2.filter(['abc','',3,4]))	

print(hasattr(f2,'blocked2'))	
setattr(f2,'blocked2','abc')	# 给对象f2新建一个属性名为blocked2，其值为abc
print(f2.blocked2)

print(hasattr(f2,'filter'))		# f2是否含有方法filter



# 输出结果
True
True
['']
['abc', 3, 4]
False
abc
True
```

#### 查询对象的所有属性及其对应值
obj.\_\_dict\_\_ ：查询对象obj的所有属性及其对应值，返回值是一个字典，key为属性名，value是属性值。
```
# 续过滤器例子
print(f1.__dict__)
print(f2.__dict__)

# 输出结果
{'blocked': [''], 'blocked2': 'abc'}
```


### dir(obj) 获悉对象的属性和方法
dir(obj)：获悉一个对象所属类的属性和方法。</br>
参数：对象</br>
返回值：包含字符串的列表</br>
注意： 类似\_\_XXX\_\_的属性在Python里有特殊用途。例如\_\_len\_\_是在使用len()时，调用对象\_\_len\_\_()的方法。</br>
```
# 以下两种写法等价
>>> len('abc')
3
>>> 'abc'.__len__()
3
>>> 
```
```
# 接过滤器例子
print(dir(f1))
print(dir(f2))

# 输出结果
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'blocked', 'filter']
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'blocked', 'filter']
```

## 接口和内省
接口（协议）interface ：对外暴露的方法和属性。</br>
Python多态不关心对象是怎么样的，只关心对象能干什么：</br>
* 获取对象属性 obj.\_\_dict\_\_</br>
* 检查方法是否存在 hasattr()</br>
* 检查方法是否可调用 callable(getattr(obj,attr,None))</br>


## 抽象基类
抽象基类：不应该被实例化的类，用于定义一些抽象方法，通过引入模块abc实现。</br>
标记为抽象方法：定义方法的上一行，添加@abstractmethod，和该方法的定义有相同缩进。</br>
```
from abc import ABC,abstractmethod		# 导入模块

class Talker(ABC):						# 创建ABC的子类Talker
    @abstractmethod					# 标记为抽象方法
    def talk(self):						# 仅有方法，被标记为抽象方法，无法实例化（会出错）
        pass

class Teacher(Talker):					# 创建Talker子类Teacher
    def talk(self):						# 重写子类的方法
        print('I am teacher.')

english_teacher=Teacher()				# 重写后可以被实例化
print(isinstance(english_teacher,Talker))	# 实例是Talker的对象
english_teacher.talk()					# 可以调用方法talk

# 输出结果
True
I am teacher.
```

## 扩展
### 关于Python实例属性和类属性的查找
http://python.jobbole.com/85100/

### API（Application Programming Interface,应用程序编程接口）
API是一些预先定义的函数，目的是提供应用程序与开发人员基于某软件或硬件得以访问一组例程的能力，而又无需访问源码，或理解内部工作机制的细节。

### 关于面向对象
* 如果函数操作一个全局变量，应将此变量作为类的属性和方法；
* 方法应重点关注实例的属性，而不过多考虑对象是怎么实现的；
* 慎用继承；
* 方法要保持精简。

### 确定类的属性和方法
* 分析需求：描述问题，程序要做什么？实现什么功能？对什么进行什么操作？
* 名词-类、形容词-属性、动词-方法
