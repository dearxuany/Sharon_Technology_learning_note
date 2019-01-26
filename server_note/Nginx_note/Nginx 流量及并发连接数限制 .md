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
