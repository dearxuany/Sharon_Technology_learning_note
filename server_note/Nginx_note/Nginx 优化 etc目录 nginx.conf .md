# Nginx 优化 /etc/nginx/nginx.conf
优化的目的是，通过调整 nginx 中的设置得到更好的性能，来应对大量客户端请求。</br>
对 nginx 进行优化，主要涉及 nginx.conf 这个文件，它保存有 nginx 不同模块的全部设置。</br>
在这个配置文件中，可以配置 nginx 的各种属性，此外还可搭配其它模块的协作优化。</br>
```
$ cd /etc/nginx                                        
$ ls                                              
conf.d          mime.types           nginx.conf       sites-enabled
fastcgi_params  naxsi_core.rules     proxy_params     uwsgi_params
koi-utf         naxsi.rules          scgi_params      win-utf
koi-win         naxsi-ui.conf.1.4.1  sites-available
```
nginx.conf 主要包括：</br>
顶层配置、events 模块、http 模块、mail 模块
```
$ vim nginx.conf
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # nginx-naxsi config
        ##
        # Uncomment it if you installed nginx-naxsi
        ##

        #include /etc/nginx/naxsi_core.rules;

        ##
          ##
        # nginx-naxsi config
        ##
        # Uncomment it if you installed nginx-naxsi
        ##

        #include /etc/nginx/naxsi_core.rules;

        ##
        # nginx-passenger config
        ##
        # Uncomment it if you installed nginx-passenger
        ##

        #passenger_root /usr;
        #passenger_ruby /usr/bin/ruby;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
                                                              91,1          98%        
```
注意：修改完 nginx.conf 后要记得 reload
```
sudo service nginx reload
```
## 顶层配置
user和pid基本没影响不用改，主要是 worker_process
```
user www-data; 
worker_processes 4;
pid /run/nginx.pid;
```
worker_process 定义了nginx在提供服务时，worker 支持的进程数量。</br>
这个优化值受到包括 CPU 内核数、存储数据的磁盘数、负载值在内的许多因素的影响。</br>
设置为 auto 将自动检测，一般可设置为系统的CPU内核数。</br>
```
# 查看cpu核心数
$ cat /proc/cpuinfo|grep processor                      [11:35:45]
processor	: 0
processor	: 1
processor	: 2
processor	: 3

$ lscpu                                                 [11:37:19]
Architecture:          x86_64
CPU \u8fd0\u884c\u6a21\u5f0f\uff1a    32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
\u6bcf\u4e2a\u6838\u7684\u7ebf\u7a0b\u6570\uff1a1
\u6bcf\u4e2a\u5ea7\u7684\u6838\u6570\uff1a  4
```
* 为何设置 worker_process 为 CPU 内核数？</br>
在一个使用分叉的服务器中，每一个客户端机连接都利用分叉创造一个子进程。父进程继续监听新的连接，同时子进程处理客户端。当客户端的请求结束时，子进程就退出了。因此分叉的进程是并行运行的，客户端之间不必互相等待。</br>
linux的进程设计能很好地利用CPU的性能，每一个任务(进程)被创建时，系统会为他分配存储空间等必要资源，然后在内核管理区为该进程创建管理节点，以便后来控制和调度该任务的执行。进程真正进入执行阶段，系统会给进程分配必要的CPU资源，这个行为被称为“调度”。除CPU而外，系统会为每个进程分配独有的存储空间，还有其他外设的可使用状态等。即是子进程和父进程一样，有它自己独立的硬件资源。</br>
由进程和线程逻辑的严谨定义上可知，每个进程都占有各自的CPU内存空间和各自的一套数据，而线程是多个线程共用同一CPU内存空间（即其所在进程中的空间）且公用全局变量。通常，当CPU个数和多线程线程数相等时，效率会比线程数多于核数的情况高，除非其中一个线程没有耗尽CPU的资源。
线程是CPU的调度单位，也即是CPU实际看到的是线程而不是进程，进程是OS上的概念而且一个进程默认启动一个进程，所以CPU还是会把单个进程当做是一个线程看待的。
单核CPU使用多线程，其实本质是时分复用，即不同时间段，不同的线程获得CPU的使用权，只是CPU切换得很快让外界看起来多个线程在单个CPU里好像是实现了并发而已，实则上因为需要频繁在线程间切换，单核CPU上使用多线程效率是不高的。</br>
当系统有一个以上CPU时，则线程的操作有可能非并发。当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)，通常出现在线程数等于或小于CPU核数的情况。当每个 CPU 核心运行一个进程的时，由于每个进程的资源都独立， CPU 核心之间切换的时候无需考虑上下文。当每个 CPU 核心运行一个线程的时，由于每个线程需要共享资源，所以这些资源必须从 CPU 的一个核心被复制到另外一个核心，才能继续运算，这占用了额外的开销。因此，在 CPU 为多核的情况下，多线程在性能上不如多进程，所以当前面向多核的服务器端编程中多为多进程而非多线程。在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。</br>

详情：</br>
[Linux 内核、进程、线程](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%86%85%E6%A0%B8%E3%80%81%E8%BF%9B%E7%A8%8B%E3%80%81%E7%BA%BF%E7%A8%8B.MD#linux%E5%A4%9A%E7%94%A8%E6%88%B7%E5%A4%9A%E4%BB%BB%E5%8A%A1%E7%8E%AF%E5%A2%83)</br>
[Python 多任务/多连接：
线程 Thread、进程 Process、协程 Coroutine](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/python_note/Python%20%E5%A4%9A%E4%BB%BB%E5%8A%A1%E3%80%81%E5%A4%9A%E8%BF%9E%E6%8E%A5%EF%BC%9A%E7%BA%BF%E7%A8%8B%20Thread%E3%80%81%E8%BF%9B%E7%A8%8B%20Process%E3%80%81%E5%8D%8F%E7%A8%8B%20Coroutine.MD)

## event 模块
```
events {
        worker_connections 768;
        multi_accept on;
}
```
worker_connections 设置一个worker进程可以同时打开的最大连接数，受 events 里面的 worker_rlimit_nofile 参数所限制。</br>
multi_accept 的作用是告诉 nginx 在收到新链接的请求通知时，尽可能接受链接，一般设置为 on。</br>

## http 模块
调优主要涉及 http 模块中前三个 settings 小模块
```
http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```
### Basic Settings
* sendfile on</br>
sendfile() 直接从磁盘上读取数据到操作系统缓冲。由于这个操作是在内核中完成的，sendfile() 比 read() 和 write() 联合使用要更加有效率，所以要打开。
* tcp_nopush on</br>
配置 nginx 在一个包中发送全部的头文件，而不是一个一个发送。
* tcp_nodelay </br>
配置 nginx 不要缓存数据，应该快速的发送小数据，通常仅用于频繁发送小碎片信息且无需立马响应的实时数据传输中。
* keepalive_timeout</br>
指定了与客户端的 keep-alive 链接的超时时间。服务器会在这个时间后关闭链接。按需设置，降低此值可避免worker等待过长时间。

### logging setings
```
access_log off;
error_log /var/log/nginx/error.log;
```
access_log 确定了 nginx 是否保存访问日志，关闭可降低磁盘IO，提升速率。</br>
error_log 设置 nginx 应当记录临界错误。
```
# 磁盘读写bi和bo数值很高则磁盘读写繁忙
 $ vmstat 1 5                                            [12:00:07]
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 1923828 516404 3095436    0    0   175    73    2    0  4  2 93  1  0
 0  0      0 1923688 516404 3095436    0    0     0    12 1321 1897  3  0 97  0  0
 0  0      0 1923688 516404 3095436    0    0     0     0 1201 1707  2  0 98  0  0
 0  0      0 1923688 516416 3095432    0    0     0   164 1195 1646  2  0 97  1  0
 0  0      0 1923688 516416 3095436    0    0     0     0  993 1349  2  1 97  0  0

```
## Gzip Settings
```
        gzip on;
        gzip_disable "msie6";

         gzip_vary on;
         gzip_proxied any;
         gzip_comp_level 6;
         gzip_buffers 16 8k;
         gzip_http_version 1.1;
         gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
```
这部分基本全部默认打开使用
* gizp on </br>
是否压缩一搬打开，提高传输速度
* gzip_disable </br> 
指定的客户端禁用 gzip 功能
* gzip_proxied </br> 
 允许或禁止基于请求、响应的压缩，设置为 any，就可以 gzip 所有的请求。
* gzip_comp_level </br> 
压缩等级，1-9，9最慢但压缩效果最好
* gzip_types</br> 
压缩类型，可按需添加


