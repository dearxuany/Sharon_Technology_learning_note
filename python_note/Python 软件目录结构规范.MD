# Python 软件目录结构规范
Python 的软件目录结构一般需要包含以下这些结构：
```
Foo/
|-- bin/
|   |-- foo
|
|-- foo/
|   |-- tests/
|   |   |-- __init__.py
|   |   |-- test_main.py
|   |
|   |-- __init__.py
|   |-- main.py
|
|-- docs/
|   |-- conf.py
|   |-- abc.rst
|
|-- setup.py
|-- requirements.txt
|-- README 
```
* bin/: 存放项目的一些可执行文件</br>
* foo/: 存放项目的所有源代码</br>
(1) 源代码中的所有模块、包都应该放在此目录。不要置于顶层目录</br>
(2) 其子目录tests/存放单元测试代码</br>
(3) 程序的入口最好命名为main.py</br>
* docs/: 存放一些文档</br>
* setup.py: 安装、部署、打包的脚本</br>
业界标准做法是用 Python 流行的打包工具 setuptools 来生成 setup.py </br>
* requirements.txt: 存放软件依赖的外部 Python 包列表</br>
可用在项目所在的 virtualenv 中只用命令 pip freeze 自动生成</br>
* README: 项目说明文件</br>

## 模块和包
### 模块
Python模块是最主要的抽象层之一，抽象层允许将代码分为不同部分，一层控制用户操作相关接口，另一层处理底层数据操作。 </br>
#### 模块命名规则</br> 
模块名称要短、使用小写，并避免使用特殊符号，比如点(.) 和问号(?)。 </br>
如果愿意您可以将模块命名为 my_spam.py，不过并不推荐在模块名中使用下划线。 </br>
在模块名称中使用其他字符（空格或连字号）将阻止导入（-是减法运算符），因此请尽量保持模块名称简单，以无需分开单词。 </br>
</br>

#### import的原理机制</br> 
具体来说，import modu 语句将寻找合适的文件，即调用当前目录下的 modu.py 文件（如果该文件存在）。 如果没有找到这份文件，Python解释器递归地在 "PYTHONPATH" 环境变量中查找该文件，如果仍没 有找到，将抛出ImportError异常。 </br>
一旦找到 modu.py，Python解释器将在隔离的作用域内执行这个模块。所有顶层语句都会被执行，包括其他的引用。方法与类的定义将会存储到模块的字典中。然后，这 模块的变量、方法和类通过命名空间暴露给调用方。 </br>
</br>

#### import语句的书写方式</br>
开发时最省力，维护时最痛苦的方式：</br>
导入某模块中所有函数，在程序中调用这些函数时，不需要在函数名前面加模块名。
```
[...]
from modu import *
[...]
x = sqrt(4)  # sqrt是模块modu的一部分么？或是内建函数么？上文定义了么？
```
开发时最费力，维护时最清晰的方式：</br>
导入模块，在程序中调用这些函数时，需要在函数名前面加模块名。
```
import modu
[...]
x = modu.sqrt(4)  # sqrt显然是属于模块modu的。
```

### 包
任意包含 \_\_init\_\_.py 文件的目录都被认为是一个Python包。
####  \_\_init\_\_.py</br> 
如果包内的模块和子包没有代码共享的需求，使用空白的 \_\_init\_\_.py 文件是正常甚至好的做法。</br>
</br>

#### 导入包中的模块</br>
pack/ 目录下的 modu.py 文件通过 import pack.modu 语句导入。</br>
该语句会在 pack 目录下寻找\_\_init\_.py 文件，并执行其中所有顶层语句。</br>
以上操作之后，modu.py 内定义的所有变量、方法和类在pack.modu命名空间中均可看到。</br>
导入深层嵌套的包可用这个方便的语法：
```
import very.deep.module as mod 
```
该语法允许使用 mod 替代冗长的 very.deep.module



