# shell function、循环、script 追踪调试
## function
shell script 中建立函数
```
function function_name() {
函数内容
}
```
注意：</br>
因为SHELL是逐行解释编译，所以被调用的函数必须写在调用代码的前面</br>
以用户输入作为判断条件时，应将变量写在被调用函数内，不然如果函数调用自己时很容易出现死循环</br>
function中的$0、$1、$2等变量和其他变量一样，要注意局部变量和全局变量的问题</br>

## 循环 loop
### 不成立-终止
```
while [condition]
do
程序主体
done
```
### 成立-终止
```
until [condition]
do
程序主体
done
```
实例：</br>
让使用者输入一个数字，程序可以由 1+2+3... 一直累加到使用者输入的数字为止。</br>
```
#!/bin/bash
# Program
#       This program let user input a number and add numbers from 1 to the number the user input.
# History
#       2018/05/20 Sharon 1st
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:/~bin/
export PATH

read -p "place input a number: " num

if (("$num" <  0)); then
        echo "Place enter an integer greater than 0."
        exit 1
else
        s=0
        t=0
        until (("$s" > "$num"))
        do
                t=$((t+s))
                s=$((s+1))
        done
        echo "add from 1 to the number you input: " $t
        exit 0
fi


$ sh math_plustest.sh
place input a number: 99
add from 1 to the number you input:  4950
$ sh math_plustest.sh
place input a number: -10
Place enter an integer greater than 0.
```

### 固定循环
#### 迭代
和 Python 中 for ... in ... 的迭代用法相同
```
for var in con1 con2 con3（或$var_all）
do
程序主体
done
第一次循环，var=con1
第二次循环，var=con2
第三次循环，var=con3
```
实例：</br>
/etc/passwd 里面以 : 来分隔，第一栏为账号名称。</br>
将 /etc/passwd 的第一栏取出，而且每一栏都以一行字符串『The 1 account is root 』来显示，那个 1 表示行数。</br>
```
#!/bin/bash

PATH=/bin:/sbin:usr/bin:usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

account_all=$(cat /etc/passwd |cut -d ':' -f1)  #执行$()中的代码，取出第一列，即用户名 

t=1
for account in $account_all
do
        echo "The $t account is $account."
        t=$((t+1))
done
```
#### 条件循环
和 C 中的 for 条件循环相似
```
for ((i==1；i<=100；i++))
do
        程序段
done
```

## shell script 追踪测试
```
sh [-nvx] script.sh
-n 不执行script，仅查询语法，语法正确无输出
-v 先输出script再执行script
-x 边输出script内容，边执行script，走查
```
