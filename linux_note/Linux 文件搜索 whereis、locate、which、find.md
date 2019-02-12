# Linux 文件搜索 whereis、locate、which、find
## whereis
whereis 常用，比较快，因为没有搜索硬盘，主要直接从数据库中查询。
whereis 只能搜索二进制文件(-b)，man 帮助文件(-m)和源代码文件(-s)。如果想要获得更全面的搜索结果可以使用 locate 命令。
```
[sunnylinux@centOSlearning ~]$ whereis sh
sh: /usr/bin/sh /usr/share/man/man1/sh.1.gz /usr/share/man/man1p/sh.1p.gz

[sunnylinux@centOSlearning ~]$ whereis python3
python3: /usr/bin/python3 /usr/local/python3
```
## locate
locate 快而全，通过“ /var/lib/mlocate/mlocate.db ”数据库查找，但数据库不是实时更新。系统会使用定时任务每天自动执行 updatedb 命令更新一次，所以有时候你刚添加的文件，它可能会找不到，需要手动执行一次 updatedb 命令（在我们的环境中必须先执行一次该命令）。
它可以用来查找指定目录下的不同文件类型，不止查找当前目录，还会查找当前目录的子目录。
```
[sunnylinux@centOSlearning ~]$ locate python3

# 部分输出：会有非常多的输出，连python3自己的各种库都会被找出来
/usr/local/python3/share/doc
/usr/local/python3/share/man
/usr/local/python3/share/doc/glances
/usr/local/python3/share/doc/glances/AUTHORS
/usr/local/python3/share/doc/glances/CONTRIBUTING.md
/usr/local/python3/share/doc/glances/COPYING
/usr/local/python3/share/doc/glances/NEWS
/usr/local/python3/share/doc/glances/README.rst
/usr/local/python3/share/doc/glances/glances.conf
/usr/local/python3/share/man/man1
/usr/local/python3/share/man/man1/glances.1
/usr/local/python3/share/man/man1/python3.1
/usr/local/python3/share/man/man1/python3.7.1
/usr/share/doc/python-setuptools-0.9.8/docs/python3.txt
/usr/share/doc/python-setuptools-0.9.8/docs/build/html/_sources/python3.txt
/usr/share/gtksourceview-3.0/language-specs/python3.lang
/usr/share/systemtap/examples/general/tapset/python3.stp
/usr/share/systemtap/tapset/python3.stp
/usr/share/vim/vim74/autoload/python3complete.vim
```
## which
which 也比较常用，为系统内建命令，常用于找某个命令的完整路径。
which 查的是环境变量 PATH 中的内容。
```
[sunnylinux@centOSlearning ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin

[sunnylinux@centOSlearning ~]$ which python3
/usr/bin/python3
[sunnylinux@centOSlearning ~]$ which sh
/usr/bin/sh
[sunnylinux@centOSlearning ~]$ which ls
alias ls='ls --color=auto'
        /usr/bin/ls
```
有些 bash 的内建指令无法用which找到
```
[sunnylinux@centOSlearning /]$ type cd
cd 是 shell 内嵌
[sunnylinux@centOSlearning /]$ type systemctl
systemctl 是 /usr/bin/systemctl
[sunnylinux@centOSlearning /]$ type history
history 是 shell 内嵌
[sunnylinux@centOSlearning /]$ type ls
ls 是 `ls --color=auto' 的别名

[sunnylinux@centOSlearning /]$ which cd
/usr/bin/cd
[sunnylinux@centOSlearning /]$ which history
/usr/bin/which: no history in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin)
```

## find
find 的用途很多，也很重要，常见格式：
```
sudo find [要搜索的目录名] -name [要搜索的内容]

[sunnylinux@centOSlearning ~]$ sudo find /etc -name ntp
/etc/selinux/targeted/active/modules/100/ntp
/etc/ntp
```
```
find [path] [option] [action]
与时间相关的命令参数：
参数	说明
-atime	最后访问时间
-ctime	最后修改文件内容的时间
-mtime	最后修改文件属性的时间

-mtime n：n 为数字，表示为在 n 天之前的“一天之内”修改过的文件
-mtime +n：列出在 n 天之前（不包含 n 天本身）被修改过的文件
-mtime -n：列出在 n 天之内（包含 n 天本身）被修改过的文件
-newer file：file 为一个已存在的文件，列出比 file 还要新的文件名
```
