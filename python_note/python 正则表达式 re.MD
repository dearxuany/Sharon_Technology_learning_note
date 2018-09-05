# python 正则表达式 re
python 中的正则表达式，由模块re来提供支持。</br>
python 的正则表达式貌似语法上和linux SHELL的正则表达式差不多，可是还有些许不一样，要注意区别。</br>
详情见：[linux 的正则表达式](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E6%96%87%E4%BB%B6%E5%90%8D%E5%91%BD%E5%90%8D%E8%A7%84%E5%88%99.MD#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)

## python 正则表达式

python 基础正则表达式
```
[]      [非任意字符]，一定有非任意字符中的其中一个，[abc]有abc其中一个
[0-9]   有0到9其中一个数字，-省略写法
[^]     [^abc]一定有一个字符属于非a，非b，非c 
^      ^the 找行首the，区别[^]
$      the$ 找行尾the
.      任意一个字符
\      转义字符，在字符串前面加r可以使字符串中的转意字符失效
*      代表重复的0到无限多个同一字符
       o* 所有字符
       oo* 至少含有一个o
       ooo* 至少很有两个o
+      代表重复的1到无限多个同一字符
       o+ 至少一个o
{m,n}   代表重复至少m到n次的同一字符
       o{3,4} 代表ooo或oooo
```
python 扩展正则表达式
```
|      或，二选一 python|perl ，也常用于子模式p(ython|erl)
()      子模式，即括号内至少有一个符合，一次搜索多个字母组合g(la|oo)d，找glad或good
()?     可选模式，即括号中的内容可以包含，也可以不包含，字母组合g(la|oo)?d，找glad、good或gd
()*     括号中内容，可重复任意多次或不存在
()+     括号中内容至少包含1次或重复任意多次
(){m,n}  括号中内容重复m到n次
[]+     由中括号内元素组成的任意多个排列组合
```
## 模块re
模块re ：</br>
python 中的正则表达式，由模块re来提供支持。</br>
可以直接使用re模块下的函数来对正则表达式进行操作，也可以先将正则表达式转变为模式对象，再将这个模式对象当做一个实例来调用re下的模式对象可用的方法search、match。

### search 和 match
re.search(pat,string) ：搜索结果不一定从第一个字符开始匹配，可是结果必须和需要搜索的字符完全匹配；</br>
re.match(pat,string) ：从开头开始搜索，模糊匹配，当开头已经找到匹配的字符之后，后面的字符就不做检查了。</br>
参数：pat 指由字符串表示的正则、string 被搜索的字符串</br>
返回值：Match object，可以将其当为布尔型使用，找到为True，找不到为None</br>
```
>>> import re
>>> str='www.python.org'

>>> re.search('py',str)
<_sre.SRE_Match object; span=(4, 6), match='py'>
>>> re.search('w',str)
<_sre.SRE_Match object; span=(0, 1), match='w'>
>>> def test():
...     if re.search('py',str):
...         print('found')
... 
>>> test()
found

# 注意search找py找到了，由于str中的py不在开头，所以match没有找到
>>> re.match('py',str)
>>> re.match('w',str)
<_sre.SRE_Match object; span=(0, 1), match='w'>
```
### compile
通过模式对象来使用此模块下的一些方法如search、match前，必须先将用字符串表示的正则表达式用re.compile转换为python解释器可识别的模式对象。</br>
</br>
re.compile(pat) ：将字符串格式的正则表达式pat转变为模式对象。</br>
参数：字符串格式的正则表达式</br>
返回值：_sre.SRE_Pattern</br>
```
>>> str='www.python.org'
>>> re.compile('[wpao]')
re.compile('[wpao]')
# 调用方法search
>>> re.compile('[wpao]').search(str)
<_sre.SRE_Match object; span=(0, 1), match='w'>
# 调用方法match
>>> re.compile('[wpao]').match(str)
<_sre.SRE_Match object; span=(0, 1), match='w'>
```

### split
re.split(pat,str,times) ：根据由字符串表示的正则表达式来分割字符串。</br>
参数：pat 字符串正则表达式、str待分割字符串、times最大分割次数</br>
返回值：列表元素输出分割后的字符串</br>
```
>>> str='Hello world! Happy birthday, Jack!'

>>> re.split('[ !,]',str)
['Hello', 'world', '', 'Happy', 'birthday', '', 'Jack', '']
>>> re.split('[! ,]+',str)
['Hello', 'world', 'Happy', 'birthday', 'Jack', '']

# 规定分割次数
>>> re.split('[! ,]+',str,maxsplit=2)
['Hello', 'world', 'Happy birthday, Jack!']

# 添加圆括号
>>> re.split('(a)',str)
['Hello world! H', 'a', 'ppy birthd', 'a', 'y, J', 'a', 'ck!']
```

### findall
re.findall(pat,str) ：寻找与正则表达式匹配的字符串并将它们作为列表元素返回。</br>
返回值：列表</br>
```
# 列出句子中的所有单词
>>> re.findall('[a-zA-Z]+',str)
['Hello', 'world', 'Happy', 'birthday', 'Jack']
```
### sub
re.sub(pat,strYouNeed,str) ：从左到右替换str中符合正则表达式指代的内容为strYouNeed。</br>
参数：pat 字符串表示正则表达式、strYouNeed 将被替换成的内容、str 将被执行替换操作的字符串</br>
返回值：替换完成后的str字符串</br>
```
>>> pat='{name}'
>>> str='Hello,{name}!'
>>> re.sub(pat,'Sharon',str)
'Hello,Sharon!'
```
### escape
re.escape(str) ：re模块中转意字符要用\\\\两个反斜杠表示，一个是re自己的，另外一个是解释器的。可是如果有一个字符串里面有很多需要被转意的字符且需要被用于正则表达式，那么就需要有很多\\\\，手动输入不符合现实，可以用escape函数来处理，自动将字符串中的特殊字符转意（加\\\\）。
```
>>> text='https://github.com/dearxuany/Sharon_Technology_learning_note'
>>> re.escape(text)
'https://github\\.com/dearxuany/Sharon_Technology_learning_note'
>>> text2='.-()\/?'
>>> re.escape(text2)
'\\.\\-\\(\\)\\\\/\\?'
```
注意以下一种和python解释器有关的情况：</br>
使用r’str’先去除掉python解释器的转意后就会被python当成普通的字符串，这样这个字符串再用于re模块中的正则表达式时就不需要再加一个\来进行多一次的转意了。有些解释器可以自动处理这样的问题，但不是每个都可以这样处理，建议还是使用r’str’在解释器层面将转意字符的问题解决掉，之后就不需要再在re层面对那些特殊字符进行处理了。
```
>>> print('Let's go!')
  File "<stdin>", line 1
    print('Let's go!')
               ^
SyntaxError: invalid syntax
>>> print('Let\'s go!')
Let's go!
>>> print(r'Let\'s go!')
Let\'s go!

>>> print('hello\nworld')
hello
world
>>> print(r'hello\nworld')
hello\nworld

# 貌似3.7也可以直接转意
>>> re.search(r'\n','hello\nworld')
<_sre.SRE_Match object; span=(5, 6), match='\n'>
>>> re.search('\\n','hello\nworld')
<_sre.SRE_Match object; span=(5, 6), match='\n'>
>>> re.search('\n','hello\nworld')
<_sre.SRE_Match object; span=(5, 6), match='\n'>

>>> str='Let\'s go!'
>>> str
"Let's go!"
>>> re.search('\'',r'Let\'s go!')
<_sre.SRE_Match object; span=(4, 5), match="'">
>>> re.search('\'','Let\'s go!')
<_sre.SRE_Match object; span=(3, 4), match="'">
```
