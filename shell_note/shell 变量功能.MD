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







