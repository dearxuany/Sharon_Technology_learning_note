# Nginx 原理与特性
## Nginx 简介
Nginx是一款高性能的HTTP和反向代理服务器软件。</br>
Nginx为性能而生,从发布以来一直侧重于高性能,高并发,低CPU内存消耗。</br>
在功能方面:负载均衡,反向代理,访问控制,热部署,高扩展性等特性又十分适合现代的网络架构。</br>
### 与 Apache 的异同
同为 HTTP 服务器软件，Nginx 常与 Apache 放在一起比较：</br>
#### 相同点
* 同是HTTP服务器软件,都采用模块化结构设计
* 支持通用语言接口,如PHP,Python等
* 支持正向代理和反向代理
* 支持虚拟主机及ssl加密传输
* 支持缓存及压缩传输
* 支持URL重写
* 模块多,扩展性强
* 多平台支持
#### 优缺点
Nginx 优势</br>
* 轻量级 安装文件小 运行时CPU内存使用率低
* 性能强 支持多核,处理静态文件效率高,内核采用的poll模型最大可以支持50K并发连接
* 支持热部署 同时启动速度快,可以在不间断服务的情况下对软件和配置进行升级
* 负载均衡 支持容错和健康检查
* 代理功能强大 支持无缓存的反向代理,同时支持IMAP/POP3/SMTP的代理
Nginx 劣势</br>
* 相比Apache 模块要少一些,常用模块都有了,而且支持LUA语言扩展功能
* 对动态请求支持不如apache
* Windows 版本功能有限 ,受限于windows的特性,支持最好的还是*unix系统

## Nginx 常用架构
### LNMP
LNMP在服务器硬件配置相同时，相对于LAMP会使用更少的CPU和内存，是小型网站，低配服务器，和VPS的福音。
### LNAMP
LNAMP是一种互补型的架构，前面介绍过，Nginx的负载均衡和反向代理配置灵活，并发能力强，处理静态资源性能强，这些特性十分适合在前端调度。缺点是处理动态资源差一些，这正是Apache的强项，所以动态资源交给Apache处理，缺点是配置复杂对服务器硬件配置要求高。
### Web调度员Nginx
当web应用发展到一定程度时，单台服务器不足以支撑业务的正常运行，为增大吞吐量往往会使用多台服务器一起提供服务，如何充分利用多台服务器的资源，就需要一个‘调度员’，这个调度员要求能高效的接收并分发请求，知道后端的服务器健康状态，要能方便的扩展和移除。此架构充分利用了Nginx的反向代理和负载均衡的优势，Nginx本身不提供web服务，而是在前端接受web请求并分发到后端服务器处理，后端服务器可以是Apache，tomcat，IIS等。

## Nginx 进程
Nginx 工作模式：</br>
* 以多进程的方式来工作，也支持多线程；
* 采用 master - worker 模式，worker 进程由 master 进程 fork 出来，由操作系统内核机制完成 worker 的分配工作；
* 采用异步非阻塞模型支持高并发。
### master 与 worker
在 master 进程里面，先建立好需要 listen 的 socket（listenfd）之后，然后再 fork 出多个 worker 进程。</br>
所有 worker 进程的 listenfd 会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有 worker 进程会在注册 listenfd 读事件前抢 accept_mutex，抢到互斥锁的那个进程注册 listenfd 读事件，然后在读事件里调用 accept 接受该连接。</br>
当一个 worker 进程在 accept 这个连接之后，就开始读取请求、解析请求、处理请求。</br>
产生数据后，再返回给客户端，最后才断开连接，这样一个完整的请求就是这样的了。</br>
#### master
master 进程主要用来管理 worker 进程。</br>
master 主要任务：</br>
接收来自外界的信号，向各 worker 进程发送信号，监控 worker 进程的运行状态，当 worker 进程退出后(异常情况下)，会自动重新启动新的 worker 进程。</br>
* 读取并验证配置信息；
* 创建、绑定及关闭套接字；
* 启动、终止 worker 进程及维护 worker 进程的个数；
* 无须中止服务而重新配置工作；
* 控制非中断式程序升级，启用新的二进制程序并在需要时回滚至老版本；
* 重新打开日志文件；
* 编译嵌入式 perl 脚本。
#### worker
对于基本的网络事件，则是放在 worker 进程中来处理了。多个 worker 进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。
一个请求，完全由 worker 进程来处理，而且只在一个 worker 进程中处理。</br>
worker 主要任务：</br>
* 接收、传入并处理来自客户端的连接；
* 提供反向代理及过滤功能；
* nginx 任何能完成的其它任务。

## Nginx 模块
### Nginx 架构介绍
Nginx的模块从结构上分为核心模块、基础模块和第三方模块：
* 核心模块： HTTP模块、EVENT模块和MAIL模块
* 基础模块： HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块
* 第三方模块： HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块及用户自己开发的模块</br>
Nginx的模块默认编译进nginx中，如果需要增加或删除模块，需要重新编译Nginx，这一点不如Apache的动态加载模块方便。如果有需要动态加载模块，可以使用由淘宝网发起的web服务器Tengine，在nginx的基础上增加了很多高级特性，完全兼容Nginx。</br>
```
shiyanlou:nginx/ $ ls -al                                             [8:13:00]
\u603b\u7528\u91cf 72
drwxr-xr-x   5 root root 4096  8\u6708 17  2016 .
drwxr-xr-x 106 root root 4096  1\u6708 26 08:07 ..
drwxr-xr-x   2 root root 4096  6\u6708  2  2016 conf.d
-rw-r--r--   1 root root  911  3\u6708  5  2014 fastcgi_params
-rw-r--r--   1 root root 2258  3\u6708  5  2014 koi-utf
-rw-r--r--   1 root root 1805  3\u6708  5  2014 koi-win
-rw-r--r--   1 root root 2085  3\u6708  5  2014 mime.types
-rw-r--r--   1 root root 5287  3\u6708  5  2014 naxsi_core.rules
-rw-r--r--   1 root root  287  3\u6708  5  2014 naxsi.rules
-rw-r--r--   1 root root  222  3\u6708  5  2014 naxsi-ui.conf.1.4.1
-rw-r--r--   1 root root 1601  3\u6708  5  2014 nginx.conf
-rw-r--r--   1 root root  180  3\u6708  5  2014 proxy_params
-rw-r--r--   1 root root  465  3\u6708  5  2014 scgi_params
drwxr-xr-x   2 root root 4096  8\u6708 17  2016 sites-available
drwxr-xr-x   2 root root 4096  8\u6708 17  2016 sites-enabled
-rw-r--r--   1 root root  532  3\u6708  5  2014 uwsgi_params
-rw-r--r--   1 root root 3071  3\u6708  5  2014 win-utf
```
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

### HTTP 模块
Nginx 本身做的工作实际很少，当它接到一个 HTTP 请求时，它仅仅是通过查找配置文件将此次请求映射到一个 locationblock，而此 location 中所配置的各个指令则会启动不同的模块去完成工作。</br>
通常一个 location 中的指令会涉及一个 handler 模块和多个 filter 模块（当然，多个 location 可以复用同一个模块）。</br>
handler 模块负责处理请求，完成响应内容的生成，而 filter 模块对响应内容进行处理。</br>
```
$ vim /etc/nginx/sites-available/default
```
### http index 模块
ngx_http_index_module </br>
定义将要被作为默认页的文件，文件的名字可以包含变量，文件以配置中指定的顺序被 nginx 检查。 
列表中的最后一个元素可以是一个带有绝对路径的文件。 

### http log 模块
ngx_http_log_module </br>

### access 模块
ngx_http_access_module </br>
此模块提供了一个简易的基于主机的访问控制，使有可能对特定 IP 客户端进行控制。规则检查按照第一次匹配的顺序，此模块对网络地址有放行和禁止的权利。

### rewrite 模块
ngx_http_rewrite_module </br>
执行 URL 重定向,允许你去掉带有恶意的 URL，包含多个参数（修改）.利用正则的匹配，分组和引用，达到目的。

### proxy 模块
ngx_http_proxy_module </br>
此模块能代理请求到其它服务器.也就是说允许你把客户端的 HTTP 请求转到后端服务器。

### upstream 模块
ngx_http_upstream_module </br>
该指令使请求被上行信道之间的基于客户端的 IP 地址分布。
