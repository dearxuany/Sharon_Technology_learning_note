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
