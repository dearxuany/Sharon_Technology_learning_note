# Nginx 使用与配置
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
### Nginx 工作原理
Nginx的模块从结构上分为核心模块、基础模块和第三方模块：
* 核心模块： HTTP模块、EVENT模块和MAIL模块
* 基础模块： HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块
* 第三方模块： HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块及用户自己开发的模块</br>

Nginx的模块默认编译进nginx中，如果需要增加或删除模块，需要重新编译Nginx，这一点不如Apache的动态加载模块方便。如果有需要动态加载模块，可以使用由淘宝网发起的web服务器Tengine，在nginx的基础上增加了很多高级特性，完全兼容Nginx。</br>
### Nginx 常用架构
#### LNMP
LNMP在服务器硬件配置相同时，相对于LAMP会使用更少的CPU和内存，是小型网站，低配服务器，和VPS的福音。
#### LNAMP
LNAMP是一种互补型的架构，前面介绍过，Nginx的负载均衡和反向代理配置灵活，并发能力强，处理静态资源性能强，这些特性十分适合在前端调度。缺点是处理动态资源差一些，这正是Apache的强项，所以动态资源交给Apache处理，缺点是配置复杂对服务器硬件配置要求高。
#### Web调度员Nginx
当web应用发展到一定程度时，单台服务器不足以支撑业务的正常运行，为增大吞吐量往往会使用多台服务器一起提供服务，如何充分利用多台服务器的资源，就需要一个‘调度员’，这个调度员要求能高效的接收并分发请求，知道后端的服务器健康状态，要能方便的扩展和移除。此架构充分利用了Nginx的反向代理和负载均衡的优势，Nginx本身不提供web服务，而是在前端接受web请求并分发到后端服务器处理，后端服务器可以是Apache，tomcat，IIS等。

## LNMP 安装配置
### Nginx 的安装
#### 在 ubuntu 下的安装
* apt-get 直接安装
安全策略一般禁用root，所以用sudo
```
sudo apt-get update
sudo apt-get install -y nginx
```

* 源码安装</br>
ubuntu 默认的策略是什么库都不安装，经过上面的库依赖解决，可以从中了解到 nginx 依赖的库有哪些，并且可以定制安装组件或者不安装组件，开机启动或开机不启动等等。切到 /usr/local/src 到 nginx 源下载最新解压编译安装。</br>
[CentOS下的 nginx 源码安装](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%AE%89%E8%A3%85Nginx.MD)

#### 常用 nginx 命令
配置文件目录
```
/etc/init.d/nginx
```
启动 nginx (以下二选一)
```
sudo /etc/init.d/nginx start
sudo service nginx start
```
启动后，在浏览器进行测试
```
http://localhost
```
关闭 nginx
```
sudo /etc/init.d/nginx stop
sudo service nginx stop
```
重启 nginx
```
sudo /etc/init.d/nginx restart
sudo service restart
```
#### 简单配置
此处配置 LNMP 架构，P 为 PHP
```
$ cd /etc/nginx                                          [4:24:08]
$ ls -al                                             [4:33:03]
\u603b\u7528\u91cf 72
drwxr-xr-x   5 root root 4096  1\u6708 24 04:05 .
drwxr-xr-x 106 root root 4096  1\u6708 24 03:59 ..
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
drwxr-xr-x   2 root root 4096  1\u6708 24 04:05 sites-available
drwxr-xr-x   2 root root 4096  8\u6708 17  2016 sites-enabled
-rw-r--r--   1 root root  532  3\u6708  5  2014 uwsgi_params
-rw-r--r--   1 root root 3071  3\u6708  5  2014 win-utf
```
修改文件，释放模块 location ~ .php$ {}
```
sudo vim /etc/nginx/sites-available/default
```
删掉前面的注释，添加 root /usr/share/ngnix/html; 和  fastcgi_param SCRIPT_FILENAME $document_root $fastcgi_script_name; 两行
```
   location ~ \.php$ {
                root /usr/share/nginx/html;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

                # With php5-cgi alone:
                # fastcgi_pass 127.0.0.1:9000;
                # With php5-fpm:             
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
```
测试配置文件
```
 $ sudo nginx -t                            [4:48:52]
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
使配置文件生效
```
 $ sudo service nginx reload                [4:52:38]
 * Reloading nginx configuration nginx   
```
### php 安装测试
在 LNMP 中的作用或角色：nginx 本身不能处理 PHP，它只是个 web 服务器，当接收到请求后，如果是 php 请求，则发给 php 解释器处理，并把结果返回给客户端.php-fpm 是一个守护进程（FastCGI 进程管理器）用于替换 PHP FastCGI 的大部分附加功能，对于高负载网站是非常有用的。
####　安装　php5
```
sudo apt-get install -y php5-fpm
```
#### 配合 nginx 一起测试
进入/usr/share/nginx/html，该文件夹就是 nginx 配置文件里location ~ .php$ {} 中第一行所指出那个文件夹
```
$ cd /usr/share/nginx/html                               [5:10:35]
$ sudo vim phpinfo.php   
```
phpinfo.php 内容为
```
<?php
    phpinfo();
?>
```
启动 php
```
$ sudo service php5-fpm start  
```
浏览器输入访问以下页面，会看到php的信息页
```
http://localhost/phpinfo.php 
```
页面上可以看到，现时没有任何 mysql 的启动信息
####  让 php5 支持 mysql
此处省略 mysql 的安装，安装 php5-mysql 模块，然后重启php
```
$ sudo apt-get install php5-mysql
$ sudo service php5-fpm start  
```
之后查看 http://localhost/phpinfo.php 会看到 mysql 的相关信息



