# Python序列：元组
元组 tuple :</br>
支持序列基本操作，支持元素赋值、删除元素、切片赋值。</br>
不可被修改，用逗号分隔，圆括号括起。
## 创建元组
```
>>> 2,5,6
(2, 5, 6)
>>> 2
2
>>> (2)
2
>>> 2,
(2,)                      #长度为1的元组，元素值后面也要加逗号
>>> (2,)
(2,)
>>> ()                    #空元组
()
```
```
>>> (44+2)*3
138
>>> (44+2,)*3             #逗号会改变表达式的值
(46, 46, 46)
```

## tuple() 类型转换
将序列作为参数转换为元组
```
>>> lst=tuple('hello_world')      #将字符串转换为元组
>>> print(lst)
('h', 'e', 'l', 'l', 'o', '_', 'w', 'o', 'r', 'l', 'd')
```
## 元组基本操作
和列表基本相同，元组的切片也是元组，但不支持count等列表方法。</br>
详见</br> 
* 关于序列基本操作</br>
[Python序列：序列类型、通用基本操作](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E5%BA%8F%E5%88%97%EF%BC%9A%E5%BA%8F%E5%88%97%E7%B1%BB%E5%9E%8B%E3%80%81%E9%80%9A%E7%94%A8%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C.MD)
* 关于元素赋值、删除元素、切片赋值</br>
[Python序列：列表](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E5%BA%8F%E5%88%97%EF%BC%9A%E5%88%97%E8%A1%A8.MD)
```
lst=('h', 'e', 'l', 'l', 'o', '_', 'w', 'o', 'r', 'l', 'd')
>>> lst[3:6]
('l', 'o', '_')
```
