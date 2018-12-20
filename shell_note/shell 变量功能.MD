# shell 变量功能
## bash 变量规则
* = 赋值
* 两边均不支持空格
* 可为数字或英文，变量名不可以数字开头
* 赋值有空格时，用（）括起
* 对特殊符号$或\等，‘’为纯文本，“”为可执行的变量，必须成对使用
* 变量中有其他命令传递的结果，用$或``(键盘1旁边)来嵌套命令
* 约定系统变量为大写，自定义变量为小写
* export PATH将变量变为环境变量

## 变量操作
### echo
#### 查询变量值
```
$ echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin
$ echo ${PATH}
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin
$ echo $HOME
/home/sunnylinux
$ echo $MAIL
/var/spool/mail/sunnylinux
```
注意：echo时有$和没$的区别，没$是直接输出字符串了
```
$ echo aaa
aaa
$ aaa='hello'
$ echo $aaa
hello
```
#### 变量赋值
变量没赋值前默认为空
```
$ echo $name

$ name=Sharon
$ echo $name
Sharon
```
变量两边不能有空格，有空格会导致误判，字符串最好还是加引号括起
```
$ name=Sharon Li
bash: Li: 未找到命令...
$ name='Sharon Li'
$ echo $name
Sharon Li
```
注意单引号''和双""的区别，单引号''内的是单纯的字符串，双引号会被识别为变量或命令
```
$ echo $LANG
zh_CN.UTF-8
$ var='lang is $LANG'
$ echo $var
lang is $LANG
$ var2="lang is $LANG"
$ echo $var2
lang is zh_CN.UTF-8
```
可用()括起优先要执行的嵌套命令
```
$ version=$(uname -r)
$ echo $version
3.10.0-862.2.3.el7.centos.plus.i686
```

### unset
取消变量内容
```
$ echo $version
3.10.0-862.2.3.el7.centos.plus.i686
$ unset version
$ echo $version

```

## 环境变量
环境变量（全局变量），默认使用大写字母表示，自定义变量用小写字母表示</br>
常见默认环境变量：</br>
* HOME 当前用的主文件夹
* SHELL shell类型
* HISTSIZE 命令历史记录条数
* MAIL 邮件系统位置
* PATH 执行文件查找路径,:为分隔，有顺序
* LANG 语系
* RANDOM 随机数 默认0-32767 （无法被自定义）

### export 
显示环境变量
```
$ export
declare -x COLORTERM="truecolor"
declare -x DBUS_SESSION_BUS_ADDRESS="unix:abstract=/tmp/dbus-3peQcQ17tH,guid=e2468cbbe87c7e4bd7aced7c5c1b06b9"
declare -x DESKTOP_SESSION="gnome-classic"
declare -x DISPLAY=":0"
#省略大量输出

$ env
XDG_VTNR=1
SSH_AGENT_PID=2320
XDG_SESSION_ID=4
HOSTNAME=centOSlearning.SharonLi
#省略大量输出
```
环境变量声明
```
#装python3时将/usr/local/python3/bin加入PATH
PATH=$PATH:/usr/local/python3/bin/python3    #设置环境变量
export PATH
```

### 一些特殊变量
#### 语系变量
查看语系
```
$ locale
LANG=zh_CN.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=
```
语系文件目录 /usr/lib/locale/</br>
常用中文语系：</br>
```
zh_CN.big5(大五码 繁体)
zh_CN.utf8（Unicode万国码 繁简体）
zh_CN.gb2312(简体)
```
常用英文语系： 
```
en_US.utf8
```
#### PID 进程号
输出本SHELL的PID
```
$ echo $$
3179
```
#### 上个执行命令的回传码
? 表示上个执行命令的回传码，0为执行成功，非0为执行不成功
```
$ echo $$
3179
$ echo $?
0
$ name=sharon li
bash: li: 未找到命令...
$ echo $?
127

```

## 变量键盘读取
read [-tp] variable</br>
-p 加注释提示输入，不加默认空白行等待输入</br>
-t 设置等待时间，单位为秒，默认一直等待</br>
```
$ read -p 'Place input your name: ' name
Place input your name: sharon
$ echo $name
sharon
```

## 变量类型声明
bash中变量类型</br>
* 默认为char 字符串 
* 默认仅能达到整数类型，没有浮点型

### declare
declare [-airx] variable </br>
-a array 数组 </br>
-i int 整形 </br>
-x 环境变量 +x 取消环境变量</br>
-r read only 不可被更改内容，也不能重设（设置后要注销再登录才能复原变量类型）</br>
-p 查看变量类型</br>
取消声明将-改为+
```
$ sum=6+2+9
$ echo $sum
6+2+9
$ declare -i sum=6+2+9
$ echo $sum
17

```

## 变量表达式与通配符
### shell 通配符
```
*      0到无限个任意字符
？     一个？代表一个任意字符
[]    [非任意字符]，一定有非任意字符中的其中一个，[abc]有abc其中一个
[0-9]  有0到9其中一个数字，-省略写法
[^]    [^abc]有一个字符属于非a，非b，非c 
```
实例
```
含有 *[0-9]*  
开头为 *[0-9]
结尾为 [0-9]*
中间有一个字符不为 *[^a-z]*
```

### 变量表达式
变量表达式和通配符配合使用，可用于快速修改变量内容输出，但不改变变量本身的值
#### 删除
echo ${变量名#关键字}
注意：不会修改原变量的值
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/shell_images/shell%20%E5%8F%98%E9%87%8F%E5%8A%9F%E8%83%BD1.png)
```
$ var=abcabcadc12367
$ echo $var
abcabcadc12367
$ echo ${var#a*c}
abcadc12367
$ echo $var
abcabcadc12367
$ echo ${var##a*c}
12367
$ echo ${var%c*7}
abcabcad
$ echo ${var%%c*7}
ab
$ echo $var
abcabcadc12367
$ echo ${var#ab*a}
bcadc12367
$ echo ${var#ab*c}
abcadc12367
$ echo ${var##ab*c}
12367
```
#### 替换
echo ${变量名/旧字符/新字符}
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/shell_images/shell%20%E5%8F%98%E9%87%8F%E5%8A%9F%E8%83%BD2.png)
```
$ echo $var
abcabcadc12367
$ echo ${var/ab/123}
123cabcadc12367
$ echo ${var//ab/123}
123c123cadc12367
$ echo $var
abcabcadc12367
```
#### 检测、替换、赋值
* 对单一变量</br>
echo 变量名=${变量名-新变量内容}</br>
检测该变量是否已被设置</br>
未被设置：变量被赋值为新变量内容</br>
已被设置：变量保存原值不变</br>
```
$ echo var=${var-hello}
var=abcabcadc12367
$ echo var3=${var3-hello}
var3=hello
$ echo $var
abcabcadc12367
$ echo $var3

```
* 对两个变量</br>
echo 变量1=${变量2-新变量内容}</br>
检测该变量2是否已被设置</br>
未被设置：变量1被赋值为新变量内容</br>
已被设置：变量1被赋值为变量2的值</br>
变量2的值始终保持不变</br>
```
$ echo $var3

$ echo var=${var3-hello}
var=hello
$ echo $var
abcabcadc12367
$ var3=world
$ echo var=${var3-hello}
var=world
```
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/shell_images/shell%20%E5%8F%98%E9%87%8F%E5%8A%9F%E8%83%BD3.png)


