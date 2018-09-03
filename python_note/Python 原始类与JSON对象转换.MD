# Python 原始类与JSON对象转换
关于JSON：[Javascript 对象：JSON](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/javascript_note/Javascript%20%E5%AF%B9%E8%B1%A1%EF%BC%9AJSON.MD)

## 模块 JSON
模块JSON ：</br>
可将python 对象转换为JSON对象，让由其他语言编写的程序也能读取这些数据。</br>
详情见 https://docs.python.org/3/library/json.html

### Python类转JSON
#### Python类转JSON对应
```
Python                                  JSON
dict                                    object
list, tuple                             array
str                                     string
int, float, int- & float-derived Enums	number
True                                    true
False                                   false
None                                    null
```

#### json.dumps(pydict) 函数
json.dumps(pydict) ：把python对像格式转变为json格式，参数是python字典。
```
#! /usr/bin/python3

import json

# python 字典
person = {
    'name':'Bob',
    'age':20,
    'tags':['js', 'web', 'mobile'],
    'city':'Beijing',
    'hasCar': True,
    'zipcode':None
}

print('python dictionary: ',person)

# python 字典转为JSON格式
json_str=json.dumps(person)

print('JSON: ',json_str)
```
```
# 输出
python dictionary:  {'name': 'Bob', 'age': 20, 'tags': ['js', 'web', 'mobile'], 'city': 'Beijing', 'hasCar': True, 'zipcode': None}
JSON:  {"name": "Bob", "age": 20, "tags": ["js", "web", "mobile"], "city": "Beijing", "hasCar": true, "zipcode": null}
```
注意：javascript对象的key是不需要引号的，而python和JSON的key都是需要引号的！


### JSON转Python
#### JSON转Python类对应
```
JSON                Python
object              dict
array               list
string              str
number (int)        int
number (real)       float
true                True
false               False
null                None
```
#### json.loads(json_str) 函数
json.loads(json_str) ：将JSON格式对象转换为python格式对象，参数为json对象。
```
# JSON格式转python对象
person2=json.loads(json_str)

print('JSON to python dictionary: ',person2)
```
```
# 输出
JSON to python dictionary:  {'name': 'Bob', 'age': 20, 'tags': ['js', 'web', 'mobile'], 'city': 'Beijing', 'hasCar': True, 'zipcode': None}
```
