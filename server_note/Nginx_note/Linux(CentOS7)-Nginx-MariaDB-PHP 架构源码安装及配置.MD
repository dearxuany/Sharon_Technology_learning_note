# Linux(CentOS7)-Nginx-MariaDB-PHP 架构 源码安装及配置
## 环境配置
### 系统平台
```
uname -r

# 输出结果
3.10.0-862.2.3.el7.centos.plus.i686
```
### 安装编译工具及库文件
```
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

### 安装 PCRE
PCRE 可让 Nginx 支持 Rewrite 功能。</br>
进入目录/src
```
[root@centOSlearning ~]# cd /usr/local/src
```
下载PCRE 安装包 
```
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
```
解压安装包
```
tar zxvf pcre-8.35.tar.gz
```
进入安装包目录
```
cd pcre-8.35
```
生成 makefile 文件，设置安装目录 --prefix=/usr/local/pcre/
```
[root@centOSlearning pcre-8.35]# ./configure --prefix=/usr/local/pcre/
```
编译与安装
```
[root@centOSlearning pcre-8.35]# make && make install
```
查看pcre版本
```
[root@centOSlearning pcre-8.35]# pcre-config --version
8.35
```
### php 和 mariaDB 安装
[Linux yum安装 php、php-fpm、php-mysql](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20yum%E5%AE%89%E8%A3%85%20php%E3%80%81php-fpm%E3%80%81php-mysql%20%20.md)</br>
[Linux 安装配置MySQL（MariaDB)](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEMySQL%EF%BC%88MariaDB%EF%BC%89.MD)

## Nginx 源码编译安装
到官网上下载合适的nginx版本源码
```
wget http://nginx.org/download/nginx-1.6.2.tar.gz
```
解压源码包
```
tar zxvf nginx-1.6.2.tar.gz
```
进入安装包目录
```
cd nginx-1.6.2
```
配置生成 makefile 文件，指定 nginx 安装路径 --prefix=/usr/local/webserver/nginx
```
[root@centOSlearning nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/pcre-8.35
```
编译
```
[root@centOSlearning nginx-1.6.2]# make
```
安装 nginx 到指定目录
```
[root@centOSlearning nginx-1.6.2]# make install
```
nginx 会被安装到 /usr/local/webserver 目录当中
```
$ pwd
/usr/local/webserver/nginx

```
nginx 目录结构
```
nginx
    ├── client_body_temp [error opening dir]
    ├── conf
    │   ├── fastcgi.conf
    │   ├── fastcgi.conf.default
    │   ├── fastcgi_params
    │   ├── fastcgi_params.default
    │   ├── koi-utf
    │   ├── koi-win
    │   ├── mime.types
    │   ├── mime.types.default
    │   ├── nginx.conf   # nginx 配置文件
    │   ├── scgi_params
    │   ├── scgi_params.default
    │   ├── uwsgi_params
    │   ├── uwsgi_params.default
    │   └── win-utf
    ├── fastcgi_temp [error opening dir]
    ├── html
    │   ├── 50x.html     # 一个error页面
    │   └── index.html   # welcome to nginx 那个页面
    │  
    ├── logs  # 存放各种日志
    │   ├── access.log
    │   ├── error.log
    │   ├── host.access.log
    │   └── nginx.pid  # 记录nginx当前主进程pid的文件
    ├── proxy_temp [error opening dir]
    ├── sbin  
    │   └── nginx  # nginx 启动脚本
    ├── scgi_temp [error opening dir]
    └── uwsgi_temp [error opening dir]

```
查看nginx版本
```
$ sudo /usr/local/webserver/nginx/sbin/nginx -v
nginx version: nginx/1.6.2
```

## nginx 配置
### 创建 Nginx 运行使用的用户 www
```
[root@centOSlearning conf]# /usr/sbin/groupadd www 
[root@centOSlearning conf]# /usr/sbin/useradd -g www www
```

### /usr/local/webserver/nginx/conf/nginx.conf 配置文件
查看原内容
```
cat /usr/local/webserver/nginx/conf/nginx.conf
```

查看cpu核心数
```
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
      1  AMD Turion(tm) II Dual-Core Mobile M520
```
nginx.conf 基础配置
```
cd /usr/local/webserver/nginx/conf
vim nginx.conf
```
```
user  www;  # 用户名
worker_processes  1;  # 配置为CPU核数

error_log  logs/error.log;  # 设置 error.log 错误日志路径
error_log  logs/error.log  notice;  # 错误日志级别
error_log  logs/error.log  info;

pid        logs/nginx.pid;  # 设置nginx.pid文件路径，该文件记录当前运行的 nginx主进程的pid

events {
    worker_connections  1024;  # 单核CPU默认的并发数（一般默认是1024）
}

# http{}内的内容指的是所有server的全局配置
http {
    include       mime.types;
    default_type  application/octet-stream;

    # 修改log格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;  # 设置 access.logs 路径，格式为 main，可在上方  log_format 设置

    sendfile        on;
    tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;   # 优化选项，调小限制每个请求占用worker进程的时间

    gzip  on;
    
 # 虚拟主机的配置
 server {
        listen       80;  # 监听端口号
        server_name  localhost;   # 可设为ip或url

        #charset koi8-r;

        access_log  logs/host.access.log  main;  # 虚拟主机即当前server的 access.log
        
        # location 用于指定在找文件的路径
        # 浏览器地址栏输入会和 location 后面的 pattern 作匹配，配上了可以进入location下的目录找文件 
        # location pattern {} 一般匹配
        location / {
            root   html;  # 到哪个目录下找文件，此处没有写绝对路径表示/html 是 nginx.conf 所在目录的平级目录
            index  index.html index.htm;  # 默认首页，优先级由前到后
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;   # 错误返回
        
        # location = pattern {} 为精确匹配
        location = /50x.html {
            root   html;
        }

        # 反向代理，将请求转给其他服务器，配合location的过滤实现动静分离
        # 要注意remote_addr的问题，可用 proxy_set_header 来解决
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #  
        #location ~ \.php$ {
        #    proxy_set_header X-real-ip $remote_addr; # 让下一跳的服务器不要get到这个nginx的remote_addr，而通过get header获得真实的用户IP
        #    proxy_pass   http://127.0.0.1;  # http://代理服务器ip:端口号
        #}

        # 使用 FastCG 让 nginx 支持 php
        # location ~ pattern {} 正则匹配，匹配所有以.php结尾的请求
        # 进去此分支后，如果没找到文件会显示 file not found，而不是404
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ \.php$ {
             root           html;  # 网站所在根目录
             fastcgi_pass   127.0.0.1:9000;  # 指定了fastcgi进程侦听的端口,nginx就是通过这里与php交互
             fastcgi_index  index.php; 
             fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include        fastcgi_params;
         }


        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```
### 检验配置文件正确性
```
[root@centOSlearning conf]# /usr/local/webserver/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/webserver/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/webserver/nginx/conf/nginx.conf test is successful
```

### 启动Nginx
```
[root@centOSlearning conf]# /usr/local/webserver/nginx/sbin/nginx
```
可以看到开启主进程的是root，开启worker进程的是配置文件最开头user设置的www用户
```
$ ps -aux|grep nginx
root      7042  0.0  0.0   8128   796 ?        Ss   00:31   0:00 nginx: master process ./sbin/nginx
www       7043  0.0  0.1   8320  1348 ?        S    00:31   0:00 nginx: worker process
sunnyli+  7065  0.0  0.0   6704   880 pts/1    S+   00:41   0:00 grep --color=auto nginx
```
注意：如果出现各种端口被占用，可能是之前nginx已经被启动，ps aux|grep nginx 查进程，然后杀掉nginx再重启
```
# 快速关闭 nginx，在nginx.pid查当前nginx主进程的pid，然后用kill杀了它
$ cat ./logs/nginx.pid
5913
$ sudo kill -TERM 5913
```
注意：其实如果只是修改了配置文件貌似是不用关闭重启 nginx 的，可以直接 reload 一下让配置生效，但是加模块的话还是要重启
```
/usr/local/webserver/nginx/sbin/nginx -s reload
```

## 访问站点
查看服务器ip地址
```
[root@centOSlearning conf]# ifconfig -a
```
对应 inet后面的ip，输入到浏览器中，看见welcome to nginx！表示配置成功
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/Nginx%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.png)
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/lnmp.png)
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/centos7-nginx-mariadb-php.png)
注意：非本机访问要注意防火墙的设置
```
systemctl start firewalld  # 启用防火墙
systemctl status firewalld  # 查看防火墙状态
systemctl stop firewalld  # 防火墙关闭
```
注意：如果出现 403 forbidden ，则是文件及其所在目录的权限问题，文件和所在所有父目录的权限都应被设为755

## 其他配置测试
在 http 中加一个 server
```
 server {
        listen  1234;  # 设置当前server的监听端口为1234，页面访问时的格式为 https://服务器的ip:1234
        server_name  localhost;
        access_log  logs/monitor.access.log  main;   # 本server的自定义access.log

        # http://192.168.137.100:1234
        location / {
            root  html;  # 到和/conf平级的/html目录下找文件，写绝对路径会比较好理解
            index  nginx_first.html;  # 首页设置
            }
        
        # http://192.168.137.100:1234/locationTest00.html
        # location 使用正则匹配，即是检测到 http://192.168.137.100:1234/location 这个部分就进入到这个分支了
        location ~ location {
            # 当用户IP在remote_addr中时，显示401错误，拒绝访问
            # if ($remote_addr = 192.168.137.1){
            #    return 401;
            # }
            
            # 当用户的浏览器为chrome时，所有请求都会被重写到 locationRewrite.html 页面，~* 忽略大小写
            if ($http_user_agent ~* windows) {
                rewrite ^.*$ /locationRewrite.html;  # 这个html也在本location指定的root目录下
                break;  # break必须要加！不加逻辑上同一个用户会在这个页面内一直出不去，服务器会报错
            }
            
            root /usr/local/webserver/nginx/html/nginx_location_test;
            index locationTest00.html;
            }
        
        # http://192.168.137.100:1234/get_things/ 显示首页
        # 在/usr/local/webserver/nginx/html/nginx_location_test 下，找 /get_things 目录，首页为 /get_things 下的 get_things.html
        # 如果 get_things/下有其他文件，http://192.168.137.100:1234/get_things/文件名 会下载该文件
        location /get_things/ {
            root /usr/local/webserver/nginx/html/nginx_location_test;
            index get_things.html;
            }
            
        # 这个写法其实和上面的写法是一样的，因为浏览器会自动补全最后的/
        # 两者的区别是有这个，访问 http://192.168.137.100:1234/get_things 是会显示 get_things.html，而不是 get_another_thing.html
        # 如果不写这个 location /get_things，访问 http://192.168.137.100:1234/get_things 是会显示404
        location /get_things {
            root /usr/local/webserver/nginx/html/nginx_location_test;
            index get_another_thing.html;
            }
    }

```
一个负载均衡的简单配置
```
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;  

    sendfile        on;
    tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;  

    gzip  on;
    
    # 设置负载均衡后端服务器节点，设权重为1:2，请求2次失败、超时时间大于30s，认为节点出问题
    upstream linuxidc {
        server 10.0.6.108:7080 weight=1 max_fail=2 fail_timeout=30s; 
        server 10.0.0.85:8980 weight=2 max_fail=2 fail_timeout=30s;
    }


    server {
        listen       70;  # 监听端口号
        server_name  localhost;   # 可设为ip或url

        #charset koi8-r;

        access_log  logs/host.access.log  main;  
        
        # 利用反向代理实现负载均衡
        location / {
            proxy_set_header X-real-ip $remote_addr; 
            proxy_pass  http://linuxidc;
            root   html; 
            index  index.html index.htm; 
        }

        # 利用反向代理实现负载均衡和动静分离，比上面的写法多个location的过滤
        #location ~ \.php$ {
        #    proxy_set_header X-real-ip $remote_addr; 
        #    proxy_pass  http://linuxidc; 
        #}
    }
}

```

