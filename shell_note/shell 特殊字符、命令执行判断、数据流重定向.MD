# shell 特殊字符、命令执行判断、数据流重定向
## shell 特殊字符
```
#   注释
\   转意字符，将特殊字符还原成一般字符（如\$和换行输入）
|   管线
;   连续命令分隔符（例如C中的;）
~   使用者家目录（文件名没事不要用这符号）
$   取用变量前导符
&   命令后台工作
!   非
/   目录符号
>   数据流导向，输出，“替换”
>>  数据流导向，输出，“累加”
<   数据流导向，输入 
<<  数据流导向，输入 
‘’不具变量置换功能
“”具有变量置换功能
``  可优先执行命令$()
()  子SHELL起始和结束
{}  中间命令块组合
```

## 命令执行判断
根据$?决定是否执行下一命令</br>
$?=0 成功执行，$?=非0 执行失败</br>
```
cmd1&&cmd2         cmd1的$?为0时，执行cmd2，否则不执行
cmd1||cmd2         cmd1的$?为1时，执行cmd2, 否则不执行
cmd1&&cmd2||cmd3   cmd1的$?为0时执行cmd2，为1时执行cmd3
```
## 数据流重定向
### redirect 定义
standard output与standard error output</br>
standard output：命令执行成功时回传的信息</br>
standard error output：命令执行失败时回传的信息</br>
以SHELL作为参考对象，文件信息到SHELL用输入，SHELL到文件为输出</br>
### 传送用特殊字符
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/shell_images/shell%20%E6%95%B0%E6%8D%AE%E9%87%8D%E5%AE%9A%E5%90%91.png)
