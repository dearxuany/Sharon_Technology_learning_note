# Linux 内核、进程、线程
## linux 内核 kernel
内核 kernel ：</br>
linux内核本质上是一个文件，文件内记录有检测、驱动、控制硬件的程序。内核会在系统读完BIOS和加载好引导程序后被直接加载到内存中。</br>
举个简化的例子，如果玩过单片机的话会比较好理解，比较常用的80C51式通用型单片机，外搭一些电路可实现跑马灯（一串小灯珠以不同的规律亮起来）的功能，制作的时候通常会把一段C语言程序拷到单片机，通过这段C语言程序来控制单片机里的集成电路，进而达到控制单片机输出电压的效果，再搭配外围简单的电路即可实现跑马灯的不同花式显示。linux上的内核就好比单片机里的这段C语言程序，同样起到控制物理硬件的作用。实际上，linux内核也是用C语言编写的。

## 进程 process
进程 process ：</br>
Linux系统中，触发任何一个事件时，系统都会将它定义为一个进程，并给予这个进程一个ID，称为PID。同时，依据触发这个进程的用户的相关属性关系，给予这个PID一组有效的权限设置。每个进程都有三组人的权限rwx，文件所有者权限、同用户组用户权限、其他用户权限，每个文件都其对应的所有者ID（UID）和用户组ID（GID），PID就是根据UID/GID(/etc/passwd)得出的，同一程序每次触发获得的PID不是固定的。进程其实就是一个具有触发者（用户）特征的正在运行的程序program。
</br>
查看进程相关信息</br>
```
$ ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000  3291  3281  0  80   0 -  2016 do_wai pts/0    00:00:00 bash
0 R  1000  3336  3291  0  80   0 -  2423 -      pts/0    00:00:00 ps
```
bash是ps的父进程，ps其实就是执行ps -l触发的，是bash的子进程。

### 父进程和子进程
分叉 fork-and-exec ：</br>
具体可分为两步</br>
1.fork 分叉</br>
从父进程的当前状态复制出一个分支（暂存进程），这个分支的PPID（父进程ID）为父进程的PID且会有一个独立的PID，其余内容和父进程是一模一样的。</br>
2.exec 执行</br>
之后，暂存进程会以exec的方式加载实际要执行的程序，然后改变程序名，成为一个真正的子进程。</br>
</br>
一些常用的简单命令如ls -al、rm、mkdir等，它们执行完就结束了，所以PID很快就停止了。也有一些常驻进程，他们一般被称为系统服务daemon，例如crond是开机自启的，它的PID是1223。
```
$ service crond status
Redirecting to /bin/systemctl status crond.service
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 六 2018-09-15 19:27:57 CST; 23min ago
 Main PID: 1223 (crond)
   CGroup: /system.slice/crond.service
           └─1223 /usr/sbin/crond -n
```
### linux多用户、多任务环境
linux的进程设计能很好地利用CPU的性能，每一个任务(进程)被创建时，系统会为他分配存储空间等必要资源，然后在内核管理区为该进程创建管理节点，以便后来控制和调度该任务的执行。进程真正进入执行阶段，系统会给进程分配必要的CPU资源，这个行为被称为“调度”。除CPU而外，系统会为每个进程分配独有的存储空间，还有其他外设的可使用状态等。即是子进程和父进程一样，有它自己独立的硬件资源。</br>
</br>
Linux默认启动6个终端机登陆环境进程，[Alt]+[F1] X-windows [Alt]+[F2-7] tty1-tty6，可以根据实际需要调整数量。linux上，如果某个进程卡死了，可以从当前的tty转到其他tty，然后ps -aux 找出出错的进程并kill了它，再返回到原来的tty就可以了。尽管现在CPU性能已经很好，可以支持不同进程间的快速切换，但由于系统的服务会不断升级，所以总有一天还是会耗尽PC的CPU和RAM资源。</br>
</br>
在一个使用分叉的服务器中，每一个客户端机连接都利用分叉创造一个子进程。父进程继续监听新的连接，同时子进程处理客户端。当客户端的请求结束时，子进程就退出了。因此分叉的进程是并行运行的，客户端之间不必互相等待。Window是不支持分叉的，但是现代的linux/Unix就支持得很好，但是因为每个进程都要求有自己的独立内存，所以以进程的方式实现多连接会很耗费资源，所以有了线程这一概念。Win比较着重多线程，linux比较着重分叉（多进程）。</br>
</br>
## 线程  thread 或Lightweight Process
线程 thread或 Lightweight Process ：</br>
线程 thread 或 Lightweight Process，简称LWP，轻量级进程的意思。在Unix SystemV及SunOS中也被称为轻量进程（lightweight processes），但轻量进程更多指内核线程(kernel thread)，而把用户线程(user thread)称为线程。计算机科学术语，指运行中的程序的调度单位。</br>
</br>
进程是资源分配的基本单位，线程是调度的基本单位。进程和线程都可以被调度，只是线程是更小的可以调度的单位而且线程不单独占有系统的资源，而是多个线程共享一个进程中的资源。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并行多个线程，每条线程并行执行不同的任务。</br>
</br>
当每个 CPU 核心运行一个进程的时，由于每个进程的资源都独立， CPU 核心之间切换的时候无需考虑上下文。当每个 CPU 核心运行一个线程的时，由于每个线程需要共享资源，所以这些资源必须从 CPU 的一个核心被复制到另外一个核心，才能继续运算，这占用了额外的开销。因此，在 CPU 为多核的情况下，多线程在性能上不如多进程，所以当前面向多核的服务器端编程中多为多进程而非多线程。</br>
