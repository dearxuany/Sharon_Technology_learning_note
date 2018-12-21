# shell 正则表达式
## 简介
### 正则表达式 regular expression</br>
通过一些特殊字符的排列，用以查找、替换、删除一行或多行字符，实际上是个文本字符串处理工具；</br>
一般结合管道命令，如sed、awk等使用；</br>
正则表达式可分为 基础正则表达式 和 扩展正则表达式，扩展比基础更加严谨一些。</br>

### 与通配符的区别</br>
通配符是bash的一个接口功能，而正则表达式是字符串处理的一种表示方式，关于 [通配符](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/shell_note/shell%20%E5%8F%98%E9%87%8F%E5%8A%9F%E8%83%BD.MD#shell-%E9%80%9A%E9%85%8D%E7%AC%A6)

### 注意
使用正则表达式时要特别注意语系环境，不用语系对字符串的处理方式不一样，所以使用的正则表达式也可能不同，不然会出现选取错误。</br>
shell 的正则表达式貌似和 python 的有点不同，要注意区分，关于 [python 正则表达式](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%20re.MD#python-%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)

## 基础正则表达式
特殊字符表示
```
特殊符号	代表意义
[:alnum:]	代表英文大小写字符及数字，亦即 0-9, A-Z, a-z
[:alpha:]	代表任何英文大小写字符，亦即 A-Z, a-z
[:blank:]	代表空白键与 [Tab] 按键两者
[:cntrl:]	代表键盘上面的控制按键，亦即包括 CR, LF, Tab, Del.. 等等
[:digit:]	代表数字而已，亦即 0-9
[:graph:]	除了空格符 (空白键与 [Tab] 按键) 外的其他所有按键
[:lower:]	代表小写字符，亦即 a-z
[:print:]	代表任何可以被打印出来的字符
[:punct:]	代表标点符号 (punctuation symbol)，亦即：" ' ? ! ; : # $...
[:upper:]	代表大写字符，亦即 A-Z
[:space:]	任何会产生空白的字符，包括空白键, [Tab], CR 等等
[:xdigit:]	代表 16 进位的数字类型，因此包括： 0-9, A-F, a-f 的数字与字符
```
基础正则表达式字符
```
[]      [非任意字符]，一定有非任意字符中的其中一个，[abc]有abc其中一个
[0-9]   有0到9其中一个数字，-省略写法
[^]     [^abc]一定有一个字符属于非a，非b，非c 
^      ^the 找行首the，区别[^]
$      the$ 找行尾the
^$     找空白行
*      和通配符的不一样，此处代表重复的0到无限多个同一字符
       0到无限个字符
       o* 所有字符
       oo* 至少含有一个o
       ooo* 至少很有两个o
？     一个？代表一个任意字符
.      任意一个字符
.*     0个到多个字符
\      转义字符
\{2，5\}  o\{2\} 找含有两个o的字符串，o\{2，5\}找含有2-5个o的字符串
```

## 扩展正则表达式
```
|      或，用于egrep
+      o+ 至少有一个o，区别o*,相当于oo*
()     组，一次搜索多个字母组合g（la|oo）d，找glad或good
()+    多个重复组，g(xyz)+d，找g和d间有n个xyz的任意组合
```
