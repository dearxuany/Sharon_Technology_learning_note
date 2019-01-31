# Linux(ubuntu)-Nginx-MySQL-PHP 架构 apt-get 安装配置
## Nginx 安装配置
### 在 ubuntu 下的安装
* apt-get 直接安装
安全策略一般禁用root，所以用sudo
```
sudo apt-get update
sudo apt-get install -y nginx
```

* 源码安装</br>
ubuntu 默认的策略是什么库都不安装，经过上面的库依赖解决，可以从中了解到 nginx 依赖的库有哪些，并且可以定制安装组件或者不安装组件，开机启动或开机不启动等等。切到 /usr/local/src 到 nginx 源下载最新解压编译安装。</br>

### 常用 nginx 命令
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
查看 nginx 状态
```
service nginx status
```
查看错误日志
```
tail /var/log/nginx/error.log
```
注意：apt-get 安装和源码编译安装路径，源码编译安装貌似不能用 service 相关的命令
### 简单配置
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
                root /usr/share/nginx/html;   # php文件所在目录
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

                # With php5-cgi alone:
                # fastcgi_pass 127.0.0.1:9000;
                # With php5-fpm:             
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  # 设置格式，此处为 http://localhost/nginx/phpinfo
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
## php 安装测试
在 LNMP 中的作用或角色：nginx 本身不能处理 PHP，它只是个 web 服务器，当接收到请求后，如果是 php 请求，则发给 php 解释器处理，并把结果返回给客户端.php-fpm 是一个守护进程（FastCGI 进程管理器）用于替换 PHP FastCGI 的大部分附加功能，对于高负载网站是非常有用的。

### 安装 php5
```
sudo apt-get install -y php5-fpm
```
### 配合 nginx 一起测试
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
###  让 php5 支持 mysql
此处省略 mysql 的安装，安装 php5-mysql 模块，然后重启php
```
$ sudo apt-get install php5-mysql
$ sudo service php5-fpm start  
```
之后查看 http://localhost/phpinfo.php 会看到 mysql 的相关信息

## 修改端口
在某些情况下我们需要改变服务器的端口，因为套接字是通过绑定 ip 端口连接的，那就只需要将刚才的 nginx 的 default（所在目录/etc/nginx/sites-available/）配置中的端口打开，然后改成你想要监听的端口。nginx 的 http 默认端口是 80。
```
sudo vim /etc/nginx/sites-available/default
```
将端口修改为 9000 ，改完后 reload 配置文件并重启 nginx 和 php
```
server {
        listen 9000 default_server;  # 监听端口号
        listen [::]:9000 default_server ipv6only=on;

        root /usr/share/nginx/html;   # 到哪个目录去找文件
        index index.html index.htm;   # 默认首页，优先顺序有前到后

        # Make site accessible from http://localhost/
        server_name localhost; # 可改为ip地址或url

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
```
之后，页面的访问地址需要改为
```
http://localhost:9000/phpinfo.php 
```
