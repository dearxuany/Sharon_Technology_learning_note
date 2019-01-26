# Nginx 流量及并发连接数限制
Nginx 流量及并发连接数限制，主要也是修改 nginx.conf 这个文件，要注意 apt-get 安装和源码编译安装而导致的 nginx.conf 路径的区别。</br>
## 流量限制
流量限制基于 ngx_http_core_module 模块
### nginx 配置文件和要下载测试内容的路径
需要修改的 nginx 配置文件 nginx.conf 路径
```
$ pwd                                                
/usr/local/nginx/conf
$ ls                                              
fastcgi.conf            koi-win             scgi_params
fastcgi.conf.default    mime.types          scgi_params.default
fastcgi_params          mime.types.default  uwsgi_params
fastcgi_params.default  nginx.conf          uwsgi_params.default
koi-utf                 nginx.conf.default  win-utf
```
下载测试内容路径，将测试下载那个mp4文件
```
$ pwd                                      
/home/shiyanlou/Documents/seven
$ ls                                        
README.md  seven.mp4
```
### 修改 nginx 配置文件
在 http 的 server 中加一个 location，设置传输量大于3m时，传输速率开始被限制在20k以内
```
 location /seven/ {
     root /home/shiyanlou/Documents;   # 测试文件所在目录的上一层目录
     limit_rate_after 3m;  # 传输量限制
     limit_rate 20k;  # 传输速率限制
     }
```
实现流量限制由两个指令 limit_rate 和 limit_rate_after 共同完成</br>
* limit_rate_after</br>
语法：limit_rate_after size;</br>
默认值：limit_rate_after 0;</br>
作用域：http, server, location, if in location</br>
作用范围：http，server，location，if inlocation</br>
设置不限速传输的响应大小。当传输量大于此值时，超出部分将限速传送。</br>
* limit_rate</br>
语法：limit_rate rate;</br>
默认值：limit_rate 0;</br>
作用域：http, server, location, if in location</br>
限制向客户端传送响应的速率限制。参数 rate 的单位是字节/秒，设置为 0 将关闭限速。 
nginx 按连接限速，所以如果某个客户端同时开启了两个连接，那么客户端的整体速率是这条指令设置值的 2 倍。</br>
更多：http://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate

### 修改好配置文件后重启 nginx
源码编译安装，直接重启配置文件就会自动reload了，可以用tree找找当前的 ./nginx 在哪个位置
```
$ pwd                                         
/usr/local/nginx
$ sudo ./sbin/nginx        
```
### 测试下载速度
```
$ wget http://ifconfig可查到的ip地址/seven/seven.mp4    
```
## 并发连接数量限制
并发连接数量限制基于ngx_http_limit_zone_module 模块
### 修改配置文件
定义一个叫“one”的记录区，总容量为 10M，以变量 $binary_remote_addr 作为会话的判断基准（即一个地址一个会话），限制 /seven/ 目录下，一个会话只能进行一个连接。 </br>
简单点，就是限制 /seven/ 目录下，一个 IP 只能发起一个连接，多过一个，一律 503。</br>
```
http {
    limit_conn_zone   $binary_remote_addr  zone=one:10m;
    ...
    server {
        ...
        location /seven/ {
            limit_conn   one  1;
            .....
        }
```
* limit_conn_zone </br>
语法： limit_conn_zone zone_name $variable the_size</br>
默认值： no</br>
作用域： http</br>
本指令定义了一个数据区，里面记录会话状态信息。 variable 定义判断会话的变量；the_size 定义记录区的总容量。</br>
* limit_conn</br>
语法： limit_conn zone_name the_size</br>
默认值： no</br>
作用域： http, server, location</br>
指定一个会话最大的并发连接数。 当超过指定的最大并发连接数时，服务器将返回 "Service unavailable" (503)。</br>
</br>

注意：</br>
在这里使用的是 $binary_remote_addr 而不是 $remote_addr。</br>
$remote_addr 的长度为 7 至 15 bytes，会话信息的长度为 32 或 64 bytes。 </br>
而 $binary_remote_addr 的长度为 4 bytes，会话信息的长度为 32 bytes。 </br>
当 zone 的大小为 1M 的时候，大约可以记录 32000 个会话信息（一个会话占用 32 bytes）。</br>
