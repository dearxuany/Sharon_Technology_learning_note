# Python 多任务/多连接：</br>线程 Thread、进程 Process、协程 Coroutine

多任务：对于本地计算机，让CPU同时处理多项工作</br>
多连接：对于网络，让服务器同时处理多个客户端连接请求</br>
</br>
实现多任务/多连接的常见方法：</br>
1.多进程forking（分叉）</br>
2. 多线程threading</br>
3. 异步I/O asynchronous I/O ：select和poll、epoll、Twisted框架（协程：单线程的异步编程、事件驱动模型）</br>
</br>
其中，win不支持分叉，而linux会把线程看作一种特殊的进程。</br>
多进程和多线程是在以并行的方式解决多连接/多任务问题，而异步是基于单线程单进程来讨论的。比如一个物理硬件的IO接口，并行接口和串行接口，对于单个接口，数据传输方式又有同步和异步之分，接口和数据传输的方式都会影响到物理硬件间数据传输的速率。</br>
</br>
Unix/linux下，通过多进程实现多任务可以用os模块中的fork()，要实现跨平台（如在win上）的多进程可用multiprocessing模块来模拟fork，进程间通信是通过Queue、Pipes等实现的。</br>
Windows下，多线程的效率比多进程要高，所以Win上实现多任务用多线程较多。但是，在Python中可以使用多线程，但因为一些问题不能有效利用多核CPU(GIL全局锁)，例如编程比较复杂、要加锁但又要避免死锁的发生，所以其实并不适合用python来通过多线程实现多任务。</br>
</br>
在线程Thread和进程 Process中，应当优选Process，因为Process更稳定，而且Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。</br>
多任务环境中，通常是master-worker模式：</br>
master主进程/主线程负责分配任务，worker负责执行任务。</br>
多进程模式最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程。</br>
多线程模式通常比多进程快一点但不会快很多，但任何一个线程挂掉都可能直接造成整个进程崩溃，因为所有线程共享进程的内存。</br>
实际生产中，有些时候还会采用多进程和多线程的混合模式。</br>
</br>
多线程和多进程只是因为CPU在多任务情景下在任务间切换的速度非常快，所以在用户角度来看好像多个任务在同时进行而已。但无论是多进程还是多线程，任务一多，效率还是不高的。因为操作系统在切换进程或者线程时需要先保存当前执行的现场环境（CPU寄存器状态、内存页等），然后把新任务的执行环境准备好（恢复上次的寄存器状态，切换内存页等），才能开始执行。这个切换过程虽然很快，但是也需要耗费时间。当多任务的任务量达到一定程度，任务间切换消耗掉系统大部分的资源，便有可能让多进程或多线程系统瘫痪。</br>
</br>
对于单进程单线程来说，要实现多任务会采用异步的数据传输方式。现时比较新的异步I/O模型是“事件驱动模型”，让系统在在单核CPU上采用单进程模型就可以高效地支持多任务。在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。对于Python，单线程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。Python基于“事件驱动模型”的网络框架有Twisted，可在基本类的基础上通过定义专用类来定义更细致的事件，以应对服务器多连接的状况。</br>

## 进程
### 多任务
#### Linux和Mac平台上的多进程
Python以多进程 multiprocessing 的方式实现多任务主要涉及到模块os（小写！）。</br>
os模块封装了常见的系统调用，多进程主要用到的是函数fork()。</br>
fork()和普通函数的主要区别：</br>
普通函数调用一次return一次，而fork()调用一次return两次。这是因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），fork()的两个返回分别是父进程和子进程内的返回。</br>
子进程永远返回0，而父进程返回子进程的ID。因为父进程可以fork出很多个子进程，如果子进程想获得父进程的ID可用getppid()，自身的ID用getpid()。</br>
注意：win没有fork，只有Unix/linux可以，Mac据说也是可以的。</br>
</br>
关于linux进程：[Linux 内核、进程、线程](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%86%85%E6%A0%B8%E3%80%81%E8%BF%9B%E7%A8%8B%E3%80%81%E7%BA%BF%E7%A8%8B.MD#%E8%BF%9B%E7%A8%8B-process)</br>
```
#! /usr/bin/python3

import os

print('process pid (now):',os.getpid())

# 先返回父进程的返回值即子进程的pid，再返回子进程的返回值0
returnValue=os.fork()

if returnValue == 0:
    print('I\'m a subprocess. My pid is {}. My parent process is {}.'.format(os.getpid(),os.getppid()))
else:
    print('I\'m the parent process. My pid is {}. My subprocess is {}.'.format(os.getpid(),returnValue))
```
```
# 注意是先出了父进程的ID然后再出子进程的ID
process pid (now): 4780
I'm the parent process. My pid is 4780. My subprocess is 4781.
I'm a subprocess. My pid is 4781. My parent process is 4780.
```
#### 跨平台的多进程（包括Win）
multiprocessing模块是Python跨平台版本的多进程模块，它提供了一个Process类来代表一个进程对象。</br>
用multiprocessing中Process来创建的子进程能比较直观地控制其开始、等待、结束，join()用于进程间的同步。</br>
官方文档：https://docs.python.org/3.7/library/multiprocessing.html

关于__name__=’\_\_main\_\_’ ：[Python 模块](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E6%A8%A1%E5%9D%97.MD#%E5%9C%A8%E6%A8%A1%E5%9D%97%E4%B8%AD%E6%B7%BB%E5%8A%A0%E6%B5%8B%E8%AF%95%E4%BB%A3%E7%A0%81)

```
#! /usr/bin/python3

from multiprocessing import Process
import os

def subprocessCode(num):
    print('The subprocess is running. Pid:',os.getpid(),' PPid:',os.getppid())
    print('{}*{}={}'.format(num,num,num*num))

if __name__ == '__main__':
    print('The pid now： ',os.getpid())
    num=int(input('Place input a number: '))
    # 新建子进程，调用子进程要执行的代码以及传入参数
    subprocess=Process(target=subprocessCode,args=(num,))
    print('The subprocess will start...')
    # 正式开启子进程
    subprocess.start()
    subprocess.join()
    # 回到父进程
    print('The subprocess is finished.')
```
```
The pid now：  5038
Place input a number: 6
The subprocess will start...
The subprocess is running. Pid: 5039  PPid: 5038
6*6=36
The subprocess is finished.
```
#### 进程池Pool：批量创建子进程
用multiprocessing中的Pool模块可批量创建子进程，Pool()的默认大小是CPU的核数，超过此默认值会产生需要后面的子进程等待前面的子进程结束的状况。（Pool的首字母为大写！！）

* 异步非阻塞
```
#! /usr/bin/python3

from multiprocessing import Pool
import os,random,time

def timeWaitTask(name):
    print('Run task {},PID:{},PPID:{}'.format(name,os.getpid(),os.getppid()))
    start=time.time()  # 获取当前时间
    time.sleep(random.random()*3)  # 随机沉睡n秒
    end=time.time()
    print('task {} have run {} second.'.format(name,(end-start)))

if __name__ == '__main__':
    print('Parent process pid:',os.getpid())
    num=int(input('How many subprocess do you want to create?' ))
    startP=time.time()
    # 创建进程池
    p=Pool(num)
    for i in range(num):
        # 以异步非阻塞方式执行子进程代码并传入参数
        p.apply_async(timeWaitTask,args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    endP=time.time()
print('All subprocesses done. Used time:',(endP-startP))
```
```
# 异步非阻塞输出：进程池进程数设为3，迭代进程数为3
Parent process pid: 6075
How many subprocess do you want to create?3
Waiting for all subprocesses done...
Run task 0,PID:6076,PPID:6075
Run task 1,PID:6077,PPID:6075
Run task 2,PID:6078,PPID:6075
task 2 have run 0.5290088653564453 second.
task 1 have run 0.7186739444732666 second.
task 0 have run 0.8787143230438232 second.
All subprocesses done. Used time: 1.0688724517822266
```
注意p.close() 必须在p.join()前面！！ 
p.close()和 p.join()主要用于及时告诉主进程等着所有子进程运行完毕后运行剩余部分。
</br>
* 修改for循环语句为for i in range(num+1):
```
# 异步非阻塞输出：进程池进程数设为3，迭代进程数为4
Parent process pid: 5684
How many subprocess do you want to create?3
Waiting for all subprocesses done...
Run task 0,PID:5685,PPID:5684
Run task 1,PID:5686,PPID:5684
Run task 2,PID:5687,PPID:5684
task 0 have run 1.0141069889068604 second.
Run task 3,PID:5685,PPID:5684
task 2 have run 1.2155463695526123 second.
task 3 have run 1.312908411026001 second.
task 1 have run 2.806411027908325 second.
All subprocesses done. Used time: 2.984306812286377
```
注意由于迭代的进程数大于进程池设置的进程数量，所以进程3出现了等待行为，它等待进程0完成后才开始执行，然而它完成得比进程1快，因为它的睡眠时间比1短。

* 固定睡眠时间为3秒time.sleep(3) 
```
# 异步非阻塞输出：进程池进程数设为3，迭代进程数为4
Parent process pid: 6096
How many subprocess do you want to create?3
Waiting for all subprocesses done...
Run task 0,PID:6097,PPID:6096
Run task 1,PID:6098,PPID:6096
Run task 2,PID:6099,PPID:6096
task 0 have run 3.0008528232574463 second.
Run task 3,PID:6097,PPID:6096
task 1 have run 3.002493381500244 second.
task 2 have run 3.0034074783325195 second.
task 3 have run 3.00197696685791 second.
All subprocesses done. Used time: 6.27057957649231
```
* 将异步非阻塞方式改为同步方式，修改 p.apply_async(timeWaitTask,args=(i,))为 p.apply(timeWaitTask,args=(i,))
```
# 同步阻塞输出：进程池进程数设为3，迭代进程数为4
Parent process pid: 6117
How many subprocess do you want to create?3
Run task 0,PID:6118,PPID:6117
task 0 have run 3.003875732421875 second.
Run task 1,PID:6119,PPID:6117
task 1 have run 3.003229856491089 second.
Run task 2,PID:6120,PPID:6117
task 2 have run 3.003627300262451 second.
Run task 3,PID:6118,PPID:6117
task 3 have run 3.0002846717834473 second.
Waiting for all subprocesses done...
All subprocesses done. Used time: 12.214302062988281
```
可见p.apply_async() 异步非阻塞方式和p.apply() 同步阻塞方式是不一样的。</br>
p.apply_async() 异步非阻塞方式进程池内三个进程同时开始，多个进程以并行方式执行代码；</br>
p.apply() 同步阻塞方式进程池内每个进程按顺序执行且只能等上一个进程完成了才开始下一个进程，多个进程以串行方式执行代码，功能上其实和单个进程多次调用同一个函数没有什么区别。</br>
</br>
关于同步/异步、阻塞/非阻塞：[Linux IO模型：同步、异步、阻塞、非阻塞](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20IO%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%90%8C%E6%AD%A5%E3%80%81%E5%BC%82%E6%AD%A5%E3%80%81%E9%98%BB%E5%A1%9E%E3%80%81%E9%9D%9E%E9%98%BB%E5%A1%9E.MD)

#### 将外部进程作为子进程
subprocess模块可启动一个子进程并控制其输入和输出，可将一个外部进程作为子进程。</br>
call()的参数是一个列表，列表的元素是字符串。</br>
如果子进程还需要输入，则可以通过communicate()方法输入。</br>
```
#! /usr/bin/python3

import subprocess,os

print('Using python to run \'ps -l\'. pid:',os.getpid())
# 用python启动子进程执行bash命令ps -l
r = subprocess.call(['ps','-l'])
print('Exit code:',r)

Using python to run 'ps -l'. pid: 6961
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000  4372  4355  0  80   0 -  2016 do_wai pts/0    00:00:01 bash
0 S  1000  6961  4372  9  80   0 -  2932 do_wai pts/0    00:00:00 python3
0 R  1000  6962  6961  0  80   0 -  2423 -      pts/0    00:00:00 ps
Exit code: 0
```
可见这个程序的第一个进程是python3，子进程是ps且返回值为0，其父进程是python3。

#### 进程间通信
操作系统提供了很多方法供进程间的通信，Python主要通过multiprocessing下的Queue （意为“队列”）、Pipes来实现进程间的通信。</br>

##### 通过Queue来进行进程间的通信
Python中multiprocessing的Queue默认类型是FIFO先进先出，进程间可通过队列进行通信，且Queue允许多个进程放入，多个进程从队列取出对象。Queue使用mutiprocessing.Queue(maxsize)创建，maxsize表示队列中可以存放对象的最大数量。Queue其实可以被看作一个媒介，先在父进程创建Queue，然后再将这个Queue作为参数传给要进行通信的进程。其实Queue在多线程通信中也会用到。multiprocessing中的Queue是queue.Queue的一个克隆。</br>
</br>
官方文档：</br>
https://docs.python.org/3.7/library/multiprocessing.html#exchanging-objects-between-processes</br>
关于 queue：[Python 集合、堆、双端队列、队列](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E9%9B%86%E5%90%88%E3%80%81%E5%A0%86%E3%80%81%E5%8F%8C%E7%AB%AF%E9%98%9F%E5%88%97%E3%80%81%E9%98%9F%E5%88%97.MD#%E9%98%9F%E5%88%97-queue)</br>
</br>
实例：
建立两个进程，一个进程用于写入队列，另一个进程用于输出队列。
```
#! /usr/bin/python3

from multiprocessing import Queue,Process
import os

def putI(q):
    print('Process used to put items into the queue, pid:',os.getpid())
    q.put(['Sharon','Tom','Jack',26])


def printO(q):
    print('Process used to print the items of the queue, pid:',os.getpid())
    print(q.get())

if __name__ == '__main__':
    print('Parent process,pid:',os.getpid())
    q=Queue()
    pI=Process(target=putI, args=(q,))
    pO=Process(target=printO, args=(q,))
    pI.start()
    pO.start()
    pI.join()
    pO.join()
    print('Fin!')
~                   

Parent process,pid: 8393
Process used to put items into the queue, pid: 8394
Process used to print the items of the queue, pid: 8395
['Sharon', 'Tom', 'Jack', 26]
Fin!
```
##### 通过Pipe进行进程间的通信
Python中multiprocessing的Pipe也支持进程通信且可双向，但是仅支持两个进程间的通信，不支持多个进程间的通信。通过mutiprocessing.Pipe(duplex=False)创建单向管道 (默认为双向)。一个进程从Pipe一端输入对象，然后被PIPE另一端的进程接收，单向管道只允许管道一端的进程输入，而双向管道则允许从两端输入。Pipe用send()方法来传送对象，在另一端使用recv()来接收。
```
#! /usr/bin/python3

from multiprocessing import Process,Pipe

def f(conn):
    conn.send('Hello, Papa!')
    print('Father:',conn.recv())

if __name__ == '__main__':
    parentconn,subprocessconn=Pipe()
    p=Process(target=f, args=(subprocessconn,))
    p.start()
    parentconn.send('Hello, Child!')
    print('Child:',parentconn.recv())
    p.join
~             

Father: Hello, Child!
Child: Hello, Papa!
```

#### 分布式进程
分布式进程：manager object控制服务进程server process作为调度者master，通过网络proxies将任务分布到其他多个worker进程中。</br>
Python中multiprocessing的managers子模块支持把多进程分布到多台机器上。分布式进程依然需要使用Queue来进行进程间的通信，但是多了一步通过managers模块把Queue通过网络暴露出去的操作，让其他机器的进程可以通过网络访问Queue。</br>
有关managers的官方文档：https://docs.python.org/3.7/library/multiprocessing.html?highlight=managers#managers </br>

##### master进程 
创建Queue队列，一个发送队列，一个接收队列；</br>
BaseManager是managers中的一个类，要定义自己的managers要先从BaseManager创建一个子类以继承BaseManager的属性和方法；</br>
调用BaseManager的方法register()，注册之前的两Queue到网络上，QueueManager.register(typeid[, callable[, proxytype[, exposed[, method_to_typeid[, create_method]]]]])，需要注意的是typeid用于必须是字符，callable用于给typeid定义的其内容、一般用lambda函数调用之前创建的Queue，其余参数可设为None ；</br>
绑定端口，QueueManager([address[, authkey]])，地址是manager process要监听的IP和端口号，如果地址为None就随机选一个，authkey密钥用于检测连接的安全性，通常为一个字节的字符串，如果为None则调用current_process().authkey；</br>
创建子类后，调用start() 或 get_server().serve_forever()，让这个子类被开启的manager process所引用；</br>
暴露Queue到网络上，写法为manager.typeid()；</br>
放置任务；</br>
manager.shutdown() 关闭服务进程。</br>
注意：重点在于register()那一步，这一步将queue注册到网络上。对比之前单台机器的进程间通信可见，之前multiprocessing中的Queue实例化之后会直接被用到Process(target=funtionName, args=(q,)) 的args 中。而分布式的情况下会多了一层封装，每个Queue要求有一个typeid，之后再通过这个typeid来将Queue暴露到网络上，所以这大概是不能直接使用multiprocessing中的Queue而使用queue中的Queue的原因。其次，Queue的作用是用来传递任务和接收结果，每个任务的描述数据量要尽量小。</br>

###### worker进程
从BaseManager创建一个子类以继承BaseManager的属性和方法，类名要和master进程中的子类名相同；</br>
QueueManager.register('typeid')网络上注册，因为只需要从网络上获取Queue中的信息，所以只需提供typeid；</br>
QueueManager(address, authkey)输入master机器的IP、端口号和密码；</br>
m.connect()方法实现网络连接；</br>
从网络上获取Queue，写法为m.typeid()；</br>
从Queue中获取数据和任务，处理完毕后，写入数据。</br>

## 线程
### 多任务
Python的标准库提供了两个模块：\_thread和threading </br>
\_thread是低级模块，threading是高级模块，对\_thread进行了封装，一般情况下，使用threading比较多。</br>
官方文档：https://docs.python.org/3.7/library/threading.html?highlight=thread#module-threading </br>

#### 线程启动
和进程启动的方法有点相似，使用start()启动线程，线程任务完成后使用join()结束；</br>
线程任务的定义用threading.Thread(target=function_name, name=thread_name)来完成；</br>
threading.current_thread()返回现时的线程对象，可调用多种方法，如name为输出线程名；</br>
任何进程默认启动一个线程，我们把该线程称为主线程MainThread，主线程可以启动新的线程。</br>
注意：全局变量和局部变量的作用域</br>
```
#! /usr/bin/python3

import threading,time,random,os

def loop():
    loops=time.time()
    print('pid:',os.getpid())
    n = 0
    print('Running Thread {}'.format(threading.current_thread().name))
    while n < 6:
        ns=time.time()
        n=n+1
        print('Running Thread {}'.format(threading.current_thread().name))
        print('n = ',n)
        time.sleep(random.random()*3)
        ne=time.time()
        print('Using time:',ne-ns)
    loope=time.time()
    print('Thread {} ended.'.format(threading.current_thread().name))
    print('Using time:',ne-ns,'n=',n)

ms=time.time()
n=10
print('pid:',os.getpid())
print('Running Thread {}'.format(threading.current_thread().name))
# 新建线程并起名
t=threading.Thread(target=loop,name='LoopThread')
# 开启线程
t.start()
# 线程运行完毕
t.join()
me=time.time()
print('Thread {} ended.'.format(threading.current_thread().name))
print('Using time:',me-ms,'n=',n))

pid: 11293
Running Thread MainThread
pid: 11293
Running Thread LoopThread
Running Thread LoopThread
n =  1
Using time: 0.4696667194366455
Running Thread LoopThread
n =  2
Using time: 0.08289432525634766
Running Thread LoopThread
n =  3
Using time: 0.6425638198852539
Running Thread LoopThread
n =  4
Using time: 2.0970077514648438
Running Thread LoopThread
n =  5
Using time: 1.0783932209014893
Running Thread LoopThread
n =  6
Using time: 0.6027817726135254
Thread LoopThread ended.
Using time: 0.6027817726135254 n= 6
Thread MainThread ended.
Using time: 5.016706228256226 n= 10
```
#### Lock 
多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响。</br>
多线程中，所有线程共享所有变量，则任何一个变量都可以被任何一个线程修改，由于线程的调度是由操作系统决定的，当两个线程交替执行时，线程们有可能在同一时间修改同一变量，只要循环次数足够多就有可能造成数据的混乱。</br>
```
import threading,time,random,os

def loop(n):
    loops=time.time()
    print('pid:',os.getpid())
    print('Running Thread {}'.format(threading.current_thread().name))
    while n < 6:
        ns=time.time()
        n=n+1
        print('Running Thread {}'.format(threading.current_thread().name))
        print('n = ',n)
        time.sleep(random.random()*3)
        ne=time.time()
        print('Using time:',ne-ns)
    loope=time.time()
    print('Thread {} ended.'.format(threading.current_thread().name))
    print('Using time:',ne-ns,'n=',n)

def loops(n):
    for i in range(6):
        print('Running Thread {}'.format(threading.current_thread().name))
        n=n+1
        print('Thread {} ended.'.format(threading.current_thread().name))
    print('n=',n)

ms=time.time()
n=10
print('pid:',os.getpid())
print('Running Thread {}'.format(threading.current_thread().name))
# 新建线程
t1=threading.Thread(target=loop,args=(0,))
t2=threading.Thread(target=loops,args=(20,))
# 开启线程
t1.start()
t2.start()
# 线程运行完毕
t1.join()
t2.join()
me=time.time()
print('Thread {} ended.'.format(threading.current_thread().name))
print('Using time:',me-ms,'n=',n)


pid: 11782
Running Thread MainThread
pid: 11782
Running Thread Thread-1
Running Thread Thread-1
n =  1
Running Thread Thread-2
Thread Thread-2 ended.
Running Thread Thread-2
Thread Thread-2 ended.
Running Thread Thread-2
Thread Thread-2 ended.
Running Thread Thread-2
Thread Thread-2 ended.
Running Thread Thread-2
Thread Thread-2 ended.
Running Thread Thread-2
Thread Thread-2 ended.
n= 26
Using time: 1.2297306060791016
Running Thread Thread-1
n =  2
Using time: 1.575538158416748
Running Thread Thread-1
n =  3
Using time: 0.44245243072509766
Running Thread Thread-1
n =  4
Using time: 2.7796084880828857
Running Thread Thread-1
n =  5
Using time: 2.2670271396636963
Running Thread Thread-1
n =  6
Using time: 0.09868454933166504
Thread Thread-1 ended.
Using time: 0.09868454933166504 n= 6
Thread MainThread ended.
Using time: 8.424857378005981 n= 10
```

可见Thread-1执行途中，Thread-2就开始执行了。同时可以看到，包括MainThread在内的三个线程，它们各自的n值都是不一样的。在多线程环境下，每个线程都有自己的局部变量。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。但局部变量不利于函数调用，Python用ThreadLocal来解决这个问题。
```
import threading,os

def loop(n):
    global a
    print('pid:',os.getpid())
    for c in range(6):
        print('Running Thread {}'.format(threading.current_thread().name),'a=',a)
        a=a+n
        a=a-n
        print('Thread {} ended.'.format(threading.current_thread().name,'a=',a))

def loops(n):
    for i in range(2):
        loop(n)

a=0
print('pid:',os.getpid())
print('Running Thread {}'.format(threading.current_thread().name,'a=',a))
# 新建线程
t1=threading.Thread(target=loop,args=(0,))
t2=threading.Thread(target=loops,args=(20,))
# 开启线程
t1.start()
t2.start()
# 线程运行完毕
t1.join()
t2.join()
print('Thread {} ended.'.format(threading.current_thread().name),'a=',a)
```
如果这段程序像上一段那样，当Thread-1执行循环中途，Thread-2开始执行了。如果在Thread-2没有执行完的过程中，Thread-1同时执行了下一次循环，就有可能把Thread-2循环中的某个a值拿去Thread-1中用了，然后导致Thread-1中的a值发生错误。</br>
</br>
如果要确保多个进程不同时修改同一个变量，则需要加锁 threading.Lock()；</br>
当多个线程同时执行lock.acquire()时，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止；</br>
只有获得锁的那个进程可以进行change_it()操作；</br>
获得锁的线程用完后一定要释放锁，否则其余线程永远无法获得锁，则成为死线程，所以要用try...finally和lock.release()来确保锁一定会被释放。</br>
加锁确保了某段关键代码只能由一个线程从头到尾完整地执行，但由于阻止了多线程并行执行，则貌似让多线程失去了些许意义。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。</br>
```
import threading,os

def change_it(n):
    global a
    print('pid:',os.getpid())
    for c in range(6):
        print('Running Thread {}'.format(threading.current_thread().name),'a=',a)
        a=a+n
        a=a-n
        print('Thread {} ended.'.format(threading.current_thread().name,'a=',a))

def loops(n):
    for i in range(6):
        lock.acquire()
        try:
            change_it(n)
        finally:
            lock.release()


a=0
lock=threading.Lock()
print('pid:',os.getpid())
print('Running Thread {}'.format(threading.current_thread().name,'a=',a))
# 新建线程
t1=threading.Thread(target=loops,args=(0,))
t2=threading.Thread(target=loops,args=(20,))
# 开启线程
t1.start()
t2.start()
# 线程运行完毕
t1.join()
t2.join()
print('Thread {} ended.'.format(threading.current_thread().name),'a=',a)
```

由进程和线程逻辑的严谨定义上可知，每个进程都占有各自的CPU内存空间和各自的一套数据，而线程是多个线程共用同一CPU内存空间（即其所在进程中的空间）且公用全局变量。</br>
通常，当CPU个数和多线程线程数相等时，效率会比线程数多于核数的情况高，除非其中一个线程没有耗尽CPU的资源。线程是CPU的调度单位，也即是CPU实际看到的是线程而不是进程，进程是OS上的概念而且一个进程默认启动一个进程，所以CPU还是会把单个进程当做是一个线程看待的。</br>
单核CPU使用多线程，其实本质是时分复用，即不同时间段，不同的线程获得CPU的使用权，只是CPU切换得很快让外界看起来多个线程在单个CPU里好像是实现了并发而已，实则上因为需要频繁在线程间切换，单核CPU上使用多线程效率是不高的。</br>
Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁，然后每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。这个问题就导致Python的多线程因为实际上只能使用单个CPU而导致效率无法提高，大概有点本末倒置的感觉吧。</br>
总体来说，对linux和python来说，多进程会比多线程更有优势。  </br>
    
#### ThreadLocal 线程变量
ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等。threading.local()在模块Threading下，ThreadLocal可让每一个线程有自己的局部变量，同时不影响这个变量在多个线程间的多个函数中的传递。</br>
</br>
关于Python类中的属性绑定：[对比两种属性绑定方法](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%EF%BC%9A%E7%B1%BB%E3%80%81%E5%AE%9E%E4%BE%8B%E3%80%81%E5%B1%9E%E6%80%A7%E3%80%81%E6%96%B9%E6%B3%95.MD#%E5%B1%9E%E6%80%A7)

```
import threading

# 实例化一个Threadlocal对象
localStudent=threading.local()

def studentName(name):
    # localStudent实例绑定属性name
    localStudent.name=name
    getStudentName()

def getStudentName():
    # 获取name属性中的值
    stuName=localStudent.name
    print('{}: Hello,{}!'.format(threading.current_thread().name,stuName))

if __name__ == '__main__':
    print('Thread:',threading.current_thread().name)
    # 分别用两个线程调用StudentName函数
    t1=threading.Thread(target=studentName,args=('Sharon',))
    t2=threading.Thread(target=studentName,args=('Jack',))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    print('Thread {} ended.'.format(threading.current_thread().name))

Thread: MainThread
Thread-1: Hello,Sharon!
Thread-2: Hello,Jack!
Thread MainThread ended.
```

