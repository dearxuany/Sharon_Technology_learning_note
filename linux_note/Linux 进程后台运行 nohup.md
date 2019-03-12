# Linux 进程后台运行 nohup
## 进程后台运行
使用Linux服务器的时候，一般是使用终端利用SSH协议登录，如果输出很多则会阻碍当期终端的使用，所以会需要让当前命令执行的程序在后台运行并将输出存储在一个文件里。
## 命令结尾加 &
Unix/Linux下一般比如想让某个程序在后台运行，很多都是使用 & 在程序结尾来让程序自动运行。
```
sh test.sh &
```
有大量输出的情况下，最好将输出重定向至指定文件
```
sh test.sh > file.log 2>&1 &
```
0、1和2分别表示标准输入、标准输出和标准错误信息输出，可以用来指定需要重定向的标准输入或输出。</br>
\> file.log指的是标准输出重定向至file.log文件，因为默认就是标准输出，所以这里的1 > file.log中的1可以省略。</br>
2>&1指的是标准错误重定向至标准输出的位置，即file.log，因为标准输出已经被重定向至file.log里了。</br>
```
# 将输出扔掉
sh test.sh > /dev/null
```
/dev/null是一个特殊的设备文件，这个文件接受到任何数据都会被丢系，通常被称为位桶、黑洞。任何被重定向至这里的内容都会被丢弃。

## nohup命令
有时候需要在Linux上设置一个后台进程，但是当你关闭terminal之时，它会被系统kill掉，所以出现了nohup。</br>
nohup命令可以将程序以忽略挂起信号的方式运行起来，被运行的程序的输出信息将不会显示到终端，且关闭执行命令的终端后程序依然在后台运行。</br>
参考：https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/
```
# 语法
nohup command-with-options &

# 执行以上命令后会输出，按回车退出nohup界面进入正常命令行即可
$ nohup: ignoring input and appending output to 'nohup.out'
```
```
$ nohup ping www.baidu.com &
[4] 10920
$ nohup: ignoring input and appending output to ‘nohup.out’
$ ps -aux|grep 10920
sunnylinux    10920  0.0  0.0   6728   680 pts/25   S    17:25   0:00 ping www.baidu.com
sunnylinux    11001  0.0  0.0  11756   932 pts/25   S+   17:25   0:00 grep --color=auto 10920
```
可以将输出保存到log中
```
# 后台启动 jetty 然后将错误信息存在 logs/lians-dev.log 中
nohup mvn -Ptest jetty:run 2>&1 >logs/lians-dev.log &   
nohup mvn  jetty:run 2>&1 >logs/lians-dev.log &  
```
