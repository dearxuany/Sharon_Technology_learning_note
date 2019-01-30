# Linux(CentOS7) Nginx 源码安装及配置
## 系统平台
```
uname -r

# 输出结果
3.10.0-862.2.3.el7.centos.plus.i686
```

## 安装编译工具及库文件
```
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

## 安装 PCRE
PCRE 可让 Nginx 支持 Rewrite 功能。
### 进入目录/src
```
[root@centOSlearning ~]# cd /usr/local/src
```

### 下载PCRE 安装包 
```
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz

2018-07-18 19:25:44 (169 KB/s) - 已保存 “pcre-8.35.tar.gz” [1996552/1996552])
```

### 解压安装包
```
tar zxvf pcre-8.35.tar.gz
```

### 进入安装包目录
```
cd pcre-8.35
```

### 编译安装
```
[root@centOSlearning pcre-8.35]# ./configure --prefix=/usr/local/pcre/
[root@centOSlearning pcre-8.35]# make && make install
```

### 查看pcre版本
```
[root@centOSlearning pcre-8.35]# pcre-config --version
8.35
```

## Nginx 安装
### 下载Nginx
（安装pcre之后记住返回上一层目录）
```
wget http://nginx.org/download/nginx-1.6.2.tar.gz

# 下载完成
2018-07-18 19:35:52 (24.3 KB/s) - 已保存 “nginx-1.6.2.tar.gz” [804164/804164])
```
### 解压
```
tar zxvf nginx-1.6.2.tar.gz
```
### 进入安装包目录
```
cd nginx-1.6.2
```

### 编译安装
--with-pcre=pcre的源码目录
```
[root@centOSlearning nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/pcre-8.35
[root@centOSlearning nginx-1.6.2]# make && make install
```

## nginx 配置
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


### 查看nginx版本
```
# /usr/local/webserver/nginx/sbin/nginx -v
nginx version: nginx/1.6.2
```

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
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

pid        logs/nginx.pid;  # 设置nginx.pid文件路径，该文件记录当前运行的 nginx主进程的pid

events {
    worker_connections  1024;
}

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
    
 server {
        listen       80;  # 监听端口号
        server_name  localhost;   # 可设为ip或url

        #charset koi8-r;

        access_log  logs/host.access.log  main;  # 虚拟主机即当前server的 access.log

        location / {
            root   html;  # 到哪个目录下找文件，此处没有写绝对路径表示/html 是 nginx.conf 所在目录的平级目录
            index  index.html index.htm;  # 默认首页，优先级由前到后
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;   # 错误返回
        
        # 等号为精确匹配
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #  
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # 让 nginx 支持 php，此处没有设置php所以没有打开
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;  # 将端口设置为9000
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

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
在 http 中加一个 server
```
 server {
        listen  1234;  # 设置当前server的监听端口为1234，页面访问时的格式为 https://服务器的ip:1234
        server_name  localhost;
        access_log  logs/monitor.access.log  main;   # 本server的自定义access.log


        location / {
            root  html;  # 到和/conf平级的/html目录下找文件，写绝对路径会比较好理解
            index  nginx_first.html;  # 首页设置
            }
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
注意：如果出现各种端口被占用，可能是之前nginx已经被启动，ps aux|grep nginx 查进程，然后杀掉nginx再重启使用新的配置
```
# 快速关闭 nginx，在nginx.pid查当前nginx主进程的pid，然后用kill杀了它
$ cat ./logs/nginx.pid
5913
$ sudo kill -TERM 5913
```

## 访问站点
查看服务器ip地址
```
[root@centOSlearning conf]# ifconfig -a
```
对应 inet后面的ip，输入到浏览器中，看见welcome to nginx！表示配置成功
```
http://主机ip
http://主机ip:1234
```

![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/Nginx%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.png)

注意：非本机访问要注意防火墙的设置
```
systemctl start firewalld  # 启用防火墙
systemctl status firewalld  # 查看防火墙状态
systemctl stop firewalld  # 防火墙关闭
```
如果出现 403 forbidden ，则是文件及其所在目录的权限问题，文件和所在所有父目录的权限都应被设为755

