# shell 逻辑判断 
## && 和 ||
根据$?决定是否执行下一命令</br>
$?=0 成功执行，$?=非0 执行失败</br>
```
cmd1&&cmd2         cmd1的$?为0时，执行cmd2，否则不执行
cmd1||cmd2         cmd1的$?为非0时，执行cmd2, 否则不执行
cmd1&&cmd2||cmd3   cmd1的$?为0时执行cmd2，cmd2的$?为非0时执行cmd3；cmd1的$?为非0时，直接执行cmd3
```

## 测试的标志与代表意义
关于某个档名的『档案类型』判断，如 test -e filename 表示存在否
```
-e	该『档名』是否存在？(常用)
-f	该『档名』是否存在且为档案(file)？(常用)
-d	该『档名』是否存在且为目录(directory)？(常用)
-b	该『档名』是否存在且为一个 block device 装置？
-c	该『档名』是否存在且为一个 character device 装置？
-S	该『档名』是否存在且为一个 Socket 档案？
-p	该『档名』是否存在且为一个 FIFO (pipe) 档案？
-L	该『档名』是否存在且为一个连结档？
```
关于档案的权限侦测，如 test -r filename 表示可读否 (但 root 权限常有例外)
```
-r	侦测该档名是否存在且具有『可读』的权限？
-w	侦测该档名是否存在且具有『可写』的权限？
-x	侦测该档名是否存在且具有『可执行』的权限？
-u	侦测该档名是否存在且具有『SUID』的属性？
-g	侦测该档名是否存在且具有『SGID』的属性？
-k	侦测该档名是否存在且具有『Sticky bit』的属性？
-s	侦测该档名是否存在且为『非空白档案』？
```
两个档案之间的比较，如： test file1 -nt file2
```
-nt	(newer than)判断 file1 是否比 file2 新
-ot	(older than)判断 file1 是否比 file2 旧
-ef	判断 file1 与 file2 是否为同一档案，可用在判断 hard link 的判定上。 
主要意义在判定，两个档案是否均指向同一个 inode 
```
关于两个整数之间的判定，例如 test n1 -eq n2
```
-eq	两数值相等 (equal)
-ne	两数值不等 (not equal)
-gt	n1 大于 n2 (greater than)
-lt	n1 小于 n2 (less than)
-ge	n1 大于等于 n2 (greater than or equal)
-le	n1 小于等于 n2 (less than or equal)
```
判定字符串的资料
```
test -z string	判定字符串是否为 0 ？若 string 为空字符串，则为 true
test -n string	判定字符串是否非为 0 ？若 string 为空字符串，则为 false。
注： -n 亦可省略
test str1 == str2	判定 str1 是否等于 str2 ，若相等，则回传 true
test str1 != str2	判定 str1 是否不等于 str2 ，若相等，则回传 false
```
多重条件判定，例如： test -r filename -a -x filename
```
-a	(and)两状况同时成立！例如 test -r file -a -x file，则 file 同时具有 r 与 x 权限时，才回传 true。
-o	(or)两状况任何一个成立！例如 test -r file -o -x file，则 file 具有 r 或 x 权限时，就可回传 true。
!	反相状态，如 test ! -x file ，当 file 不具有 x 时，回传 true
```

## test命令
上面的&&和||常与test命令合用，用于检测文件或目录是否存在、文件的类型及权限检测、两个文件间的比较
```
test [-option] filename
```
一个用于文件存在性、类型、权限判断的 shell script</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/shell_images/shell%20test%E6%96%87%E4%BB%B6%E5%88%A4%E6%96%AD.png)
```
#!/bin/bash
# Program
#       This program lets user input a filename and check the following:
#       1.exit? 2.file/directory? 3.file permission
# History
#       2018/05/02 Sharon scripttest04
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
read -p "Place input a filename: " filename
test ! -e $filename && echo "$filename does not exit." && exit 0
test -f $filename && echo "$filename is regular file."
test -d $filename && echo "$filename is directory."
test -f $filename -o -d $filename || echo "$filename is not a regular file or directory."
test -r $filename && perm="readable"
test -w $filename && perm=$perm",writable"
test -x $filename && perm=$perm",executable"
echo "$filename's permission is $perm"
exit 0

```
## 判断符号 []
常与上面的多重条件符号合用</br>
[]可以和&&、||合用，也可以和if判断合用做条件判断</br>
* []内要用两空格括住组件
[ "$filename" == "" ]注意括号[  ]内两边的两个空格，不然编译会出错；
* 变量用双引号引起来，常量用单引号引起来；
* []内使用==而不用=
* []内没有进一步的操作仅作判断时用[  ]，而括号内还需计算机做预先一步操作的用[[  ]]
```
可以直接判断    [ "$filename" == "" ]
需先识别*       [[ "$filename" == a* ]]
需先作计算      (("$s" > "$num"))
```
==的功能在[[]]和[]中的行为是不同的,如下:
```
[[ $a == z* ]]   # 如果$a以"z"开头(模式匹配)，当a的值以z开头时，结果为true。
[[ $a == "z*" ]] # 如果$a等于z*（字符匹配），当a的值为z*时，结果为true。
[ $a == z* ]     # File globbing 和word splitting将会发生。
[ "$a" == "z*" ] # 如果$a等于z*(字符匹配)，当a的值为z*时，结果为true。
```
## 条件判断
### if ... then ...
分支结构</br>
* 单个分支
```
if [判断条件1]；then
分支1
fi
```
* 两个分支
```
if [判断条件1]; then
分支1
else
分支2
fi
```
* 多个分支
```
if [判断条件1]; then
分支1
elif [判断条件2]; then
分支2
else[判断条件3]; then
分支3
fi
```
