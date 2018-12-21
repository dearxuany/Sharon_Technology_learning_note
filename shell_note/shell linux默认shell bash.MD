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
### 查询历史命令
#### 查询历史命令
```
history [n]  
```
查询最近执行的最后n条命令，输出全部不加n；</br>
```
$ history
(省略很多输出)
  997  ls -al
  998  shutdown now
  999  type ls
 1000  type -t ls
 1001  type -a ls
 1002  type cd
 1003  type python3
 1004  type -t python3
 1005  type -a python3
 1006  echo $PATH
 1007  echo ${PATH}
 1008  echo $HOME
 1009  echo $MAIL
 1010  echo $name
 1011  echo $name=Sharon
```
!n 执行his中的第n条命令</br>
!! 执行上一条命令</br>
!al 执行最近以al开头的命令</br>
```
$ !996
find ~/.bash_history
/home/sunnylinux/.bash_history
$ !!
find ~/.bash_history
/home/sunnylinux/.bash_history
```


#### 历史记录文件
```
$ find ~/.bash_history
/home/sunnylinux/.bash_history
```
~/.bash_history 最多保存1000条记录，上面history显示的有些还未被记录的；</br>
记录在每次注销后才会被写入，新写入的数据会覆盖原有的数据；</br>
同一用户登录多个终端，每个bash都会有自己的1000条记录，所以很多人习惯只用一个bash。</br>
```
history -w  #将当前记录马上写入到~/.bash_history
```


### bash 相关配置文件
login shell：tty2-tty6 完整登录流程</br>
non-login shell：Xwindow登录后进入terminal，无需重复登录</br>
```
/etc/profile      系统配置文件（最好不要改）
/etc/bashrc       为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取
~/.bash_profile   用户个人偏好配置文件，login shell 会读
~/.bash_login     同上，优先级低于bash_profile
~/.profile        同上，优先级低于bash_login
~/.bashrc         用户自己的bash偏好文件，non-login shell 会读
~/.bash_history   命令历史记录
~/.bash_logout    用户注销时需要执行的任务可以写在这个文件里（不改只有清屏）
```
修改文件后要用source命令让配置执行，否则需要重启


### bash组合按键	
```
Ctrl + C	终止目前的命令
Ctrl + D	输入结束 (EOF)，例如邮件结束的时候；
Ctrl + M	就是 Enter 啦！
Ctrl + S	暂停荧幕的输出
Ctrl + Q	恢复荧幕的输出
Ctrl + U	在提示字符下，将整列命令删除
Ctrl + Z	『暂停』目前的命令
```
