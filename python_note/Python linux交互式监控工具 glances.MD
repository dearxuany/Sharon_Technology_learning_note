# Python linux交互式监控工具 glances
glances 的设计初衷就是在当前窗口中尽可能多的显示系统信息。</br>
glances 是一款使用 Python 开发、基于 psutil 的跨平台系统监控工具。</br>
glances 的设计初衷就是在当前窗口中尽可能多的显示系统信息。</br>
在所有的 Linux 令行工具中，它与 top 命令最相似，都是命令行交互式监控工具 但是， glances 实现了比 top 命令更齐全的监控，提供了更加丰富的功能。</br>
</br>
glances 提供的系统信息包括：
```
CPU 使用率；
内存使用情况；
内核统计信息和运行队列信息
磁盘 I/O 速度 、传输和读/写比率；
文件系统中的可用 间；
磁盘适配器；
网络 I/O 速度 、传输和读/写比率；
页面 间和页面速度；
消耗资源最多的进程；
计算机信息和系统资源
```
glances 可以在用户终端上实时显示重要的系统信息，并动态刷新内容 glances每秒钟对其进行刷新，我们也可以使用命令行参数修改刷新的频率。与dstat 相同的是，glances 以将捕获到的数据保存到文件中；而不同的是 glances 提供了 API 接口以便应用程序从 lances 取数据。</br>

## 安装 glances
直接用pip安装
```
$ pip3 install psutil  # glances基于psutil
$ pip3 install glances
```
系统首先会去/usr/bin下寻找命令，如果不在这个目录中，当然就会找不到了。</br>
这个时候我们就需要为这些找不到的命令建立一个链接文件，链接到/usr/bin下。 </br>
```
$ glances
bash: glances: 未找到命令...

# 查看环境变量中存在该命令，如果存在会显示路径
$ which glances
/usr/bin/which: no glances in (/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin)

# 查找该命令所在路径
# find / -name glances
find: ‘/run/user/1000/gvfs’: 权限不够
/usr/local/python3/bin/glances
/usr/local/python3/lib/python3.7/site-packages/glances
/usr/local/python3/lib/python3.7/site-packages/glances/outputs/static/js/components/glances
/usr/local/python3/share/doc/glances

#建立链接到/usr/bin下
# ln -s /usr/local/python3/bin/glances /usr/bin
```

## glances 的使用
```
$ glances
```
为了增加可读性， glances 会以不同的颜色表示不同的状态。
* 绿色表示性能无须做任何额外工作；
* 蓝色表示系统性能有一些小问题，用户应当开始关注系统性能；
* 紫色表示性能报 ，应当采取措施；
* 红色表示性能问题严重 应当立即处理。</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Python_note_images/python%20glances%20.png)</br>
如上面显示  High memory consumption</br>
</br>
glances 中常见的命令

```
h 显示帮助信息；
q 离开程序退出；
c 按照 CPU 实时负载对系统进程排序；
m 按照内存使用状况对系统进程排序；
i 按照 I/O 使用状况对系统进程排序；
p 按照进程名称排序；
d 显示或隐藏磁盘读写状况；
f 显示或隐藏文件系统信息；
l 分开显示每个 CPU 的使用情况
```
安装了 Bottle 这个 web 框架，还能够通过 web 浏览器显示和终端相同的监控界面。
