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

### 查看nginx版本
```
[root@centOSlearning nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -v
nginx version: nginx/1.6.2
```


## nginx配置
### 进入 nginx-1.6.2内目录conf
```
[root@centOSlearning nginx-1.6.2]# cd ./conf
[root@centOSlearning conf]# 
```

### 创建 Nginx 运行使用的用户 www
```
[root@centOSlearning conf]# /usr/sbin/groupadd www 
[root@centOSlearning conf]# /usr/sbin/useradd -g www www
```

### 替换/usr/local/webserver/nginx/conf/nginx.conf的内容
查看原内容
```
cat /usr/local/webserver/nginx/conf/nginx.conf
```

查看cpu核心数
```
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
      1  AMD Turion(tm) II Dual-Core Mobile M520
```
新建一个 nginx1.conf文件
```
cd /usr/local/webserver/nginx/conf
vim nginx1.conf
```
```
# 输入以下内容
user www www;
worker_processes 1; #设置值和CPU核心数一致
error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
pid /usr/local/webserver/nginx/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;

events
{
  use epoll;
  worker_connections 65535;
}

http
{
  include mime.types;
  default_type application/octet-stream;
  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';
  
#charset gb2312;
     
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;
     
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 60;
  tcp_nodelay on;
  
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  
  gzip on; 
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;
 
  #limit_zone crawler $binary_remote_addr 10m;
  
 #下面是server虚拟主机的配置
 server
  {
    listen 80;   #监听端口
    server_name localhost;   #域名
    index index.html index.htm index.php;
    root /usr/local/webserver/nginx/html;   #站点目录
    
      location ~ .*\.(php|php5)?$
    {
      #fastcgi_pass unix:/tmp/php-cgi.sock;
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
    {
      expires 30d;
      
  # access_log off;
    }
    
    location ~ .*\.(js|css)?$
    {
      expires 15d;
      
   # access_log off;
    }
    access_log off;
  }

}

```
保存后查看是否已修改好
```
cat nginx1.conf
```
删除原来那个文件
```
rm nginx.conf
rm：是否删除普通文件 "nginx.conf"？y
```
重命名新文件
```
rename nginx1 nginx nginx1.conf
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

## 访问站点
```
[root@centOSlearning conf]# ps -ef|grep nginx
root     21947 15607  0 21:39 pts/1    00:00:00 wget http://nginx.org/download/nginx-1.6.2.tar.gz
root     27869     1  0 22:35 ?        00:00:00 nginx: master process /usr/local/webserver/nginx/sbin/nginx
www      27871 27869  0 22:35 ?        00:00:00 nginx: worker process
www      27872 27869  0 22:35 ?        00:00:00 nginx: worker process
root     28237 22073  0 23:02 pts/1    00:00:00 grep --color=auto nginx
```
查看服务器ip地址
```
[root@centOSlearning conf]# ifconfig -a
```
对应 inet后面的ip，输入到浏览器中，看见welcome to nginx！表示配置成功

![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Linux_note_images/Nginx%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.png)

## nginx 目录结构
```
[sunnylinux@centOSlearning webserver]$ tree
.
└── nginx
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
    │   ├── nginx.conf
    │   ├── nginx.conf.default
    │   ├── scgi_params
    │   ├── scgi_params.default
    │   ├── uwsgi_params
    │   ├── uwsgi_params.default
    │   └── win-utf
    ├── fastcgi_temp [error opening dir]
    ├── html
    │   ├── 50x.html
    │   └── index.html
    ├── logs
    │   ├── error.log
    │   └── nginx_error.log
    ├── proxy_temp [error opening dir]
    ├── sbin
    │   └── nginx
    ├── scgi_temp [error opening dir]
    └── uwsgi_temp [error opening dir]

```
