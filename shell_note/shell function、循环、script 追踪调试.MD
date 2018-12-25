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
