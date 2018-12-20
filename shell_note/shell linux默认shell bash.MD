# shell linux默认shell bash
## shell
Shell是系统的用户界面，提供了用户与内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核去执行。</br>
实际上Shell是一个命令解释器，它解释由用户输入的命令并且把它们送到内核。</br>
不仅如此，Shell有自己的编程语言用于对命令的编辑，它允许用户编写由shell命令组成的程序。</br>
Shell编程语言具有普通编程语言的很多特点，比如它也有循环结构和分支控制结构等，用这种编程语言编写的Shell程序与其他应用程序具有同样的效果。</br>

## linux 默认shell:bash
### 位置
/etc/shells：系统用于shell查询，用于存储shell设置的文件</br>

### bash 功能
* 命令记录：存于.bash_history，可记1000条，为前一次登录的记录，本次登录暂存于缓存
* 命令补全：[tab]
* 命令别名：alias 命令改名
* 作业控制job control、前台foreground、后台控制 background
* 脚本 shell script
* 通配符 wildcard
* \ 将命令分为两行

### bash 内置命令查询
```
type [-tpa] name 查找执行文件
```
默认显示name的类型（内/外部命令）
```
-t   file    外部
     alias   命令别名      
     builtin 内部
-p   name为外部时，才显示完整文件名
-a   PATH变量定义的路径中，将所含name的命令找出来
```
```
$ type ls
ls 是 `ls --color=auto' 的别名
$ type -t ls
alias
$ type -a ls
ls 是 `ls --color=auto' 的别名
ls 是 /usr/bin/ls
ls 是 /bin/ls

$ type cd
cd 是 shell 内嵌

$ type python3
python3 是 /usr/bin/python3
$ type -t python3
file
$ type -a python3
python3 是 /usr/bin/python3
python3 是 /bin/python3
```

