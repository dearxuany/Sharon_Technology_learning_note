# Nginx 访问控制及 DDOS 预防
访问控制：</br>
一般网站的后台都不能让外部访问，所以要添加 IP 限制，通常只允许公司的 IP 访问。访问控制就是指只有符合条件的 IP 才能访问到这个网站的某个区域。</br>
</br>
DDOS 预防：</br>
DDOS 的特点是分布式，针对带宽和服务攻击，也就是四层流量攻击和七层应用攻击，相应的防御瓶颈四层在带宽，七层的多在架构的吞吐量。</br>
</br>
此处使用 ubuntu 的 LNMP 架构进行测试：[LNMP安装配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/server_note/Nginx_note/Linux%20Nginx%20MySQL%20PHP%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.MD)
## 访问控制
访问控制基于 ngx_http_access_module 模块，允许限制某些 IP 地址的客户端访问。</br>
ngx_http_access_module 只是很简单的访问控制，而在规则很多的情况下，使用 ngx_http_geo_module 模块变量更合适。</br>
详情： http://nginx.org/en/docs/http/ngx_http_geo_module.html
### 修改配置文件 /etc/nginx/sites-available/default
和防火墙的配置有点相似，允许访问 allow，拒绝访问 deny</br>
后面可以接单个主机IP、CIRD写法的子网地址、all、unix:，如果用unix: 指禁止 socket 的访问</br>
支持 ipv4 和 ipv6 两种写法 </br>
规则由上到下顺序匹配，直到匹配到第一条符合的规则为止</br>
```
location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```
准许192.168.3.29/24的机器访问</br>
准许10.1.20.6/16这个网段的所有机器访问</br>
准许34.26.157.0/24这个网段访问</br>
除此之外的机器不准许访问</br>
```
location / {
                allow 192.168.3.29;
                allow 10.1.0.0/16;
                allow 34.26.157.0/24;
                deny all;
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }

```
## DDOS 防范
对于七层的应用攻击，我们还是可以做一些配置来防御的。</br>
使用 nginx 的 http_limit_conn 和 http_limit_req 模块通过限制连接数和请求数能相对有效的防御。</br>
* ngx_http_limit_conn_module 可以限制单个 IP 的连接数</br>
* ngx_http_limit_req_module 可以限制单个 IP 每秒请求数</br>
### 限制单个IP的连接数
详情：[并发连接数量限制](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/server_note/Nginx_note/Nginx%20%E6%B5%81%E9%87%8F%E5%8F%8A%E5%B9%B6%E5%8F%91%E8%BF%9E%E6%8E%A5%E6%95%B0%E9%99%90%E5%88%B6%20.md#%E5%B9%B6%E5%8F%91%E8%BF%9E%E6%8E%A5%E6%95%B0%E9%87%8F%E9%99%90%E5%88%B6)
```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m; //触发条件
    ...
    server {
        ...
        location /download/ {
            limit_conn addr 1;    // 限制同一时间内1个连接，超出的连接返回503
                }
           }
     }

```
### 限制单个IP每秒请求数量
通过 ngx_http_limit_req_module 模块来限制单个IP每秒的请求数量，一旦单位时间内请求数超过限制，就会返回503错误。</br>
配置需要在两个地方设置：</br>
* nginx.conf的http段内定义触发条件，可以有多个条件</br>
* 在location内定义达到触发条件时nginx所要执行的动作</br>
#### 步骤1：修改/etc/nginx/nginx.conf
在 Basic Settings 里面加入  limit_req_zone
```
http {

        ##
        # Basic Settings
        ##
        limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s; //加入这行
        ....
        ....
```
* binary_remote_addr </br>
二进制远程地址，这个参数就些这个就好了，不需要改</br>
* zone=one:10m </br> 
定义zone名字叫one，并为这个zone分配10M内存，用来存储会话（二进制远程地址），1m内存可以保存16000会话</br>
* rate=10r/s;</br>
限制频率为每秒10个请求</br>

#### 步骤2：修改/etc/nginx/sites-available/default
```
  location ~ \.php$ {
                limit_req zone=one burst=5 nodelay;   //加入这行
                root /usr/share/nginx/html;
```
* burst=5 </br>
允许超过频率限制的请求数不多于5个</br>
假设1、2、3、4秒请求为每秒9个，那么第5秒内请求15个是允许的</br>
反之，如果第一秒内请求15个，会将5个请求放到第二秒，第二秒内超过10的请求直接503，类似多秒内平均速率限制</br>
* nodelay </br>
超过的请求不被延迟处理，设置后15个请求在1秒内处理</br>

#### 步骤3：reload 配置文件、重启nginx、测试结果
reload和重启
```
$ sudo service nginx reload                     
$ sudo service nginx restart  
```
使用命令 ab 测试
```
$ sudo apt-get install apache2-utils
$ ab -n 10 -c 10 http://localhost/phpinfo.php            [1:24:50]
```
其中-n代表每次并发量，-c代表总共发送的数量，命令表示10个请求一次并发出去。
```
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done

Server Software:        nginx/1.4.6
Server Hostname:        localhost
Server Port:            80

Document Path:          /phpinfo.php
Document Length:        221 bytes

Concurrency Level:      10  # 并发数量
Time taken for tests:   0.095 seconds   # 总请求时间
Complete requests:      10   # 总请求数量
Failed requests:        6  # 请求失败数
   (Connect: 0, Receive: 0, Length: 6, Exceptions: 0)
Non-2xx responses:      4
Total transferred:      471977 bytes
HTML transferred:       470255 bytes
Requests per second:    105.19 [#/sec] (mean)  # 平均每秒请求数
Time per request:       95.066 [ms] (mean)   # 单个请求所花费的平均时间
Time per request:       9.507 [ms] (mean, across all concurrent requests)
Transfer rate:          4848.37 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    7   4.5     10      10
Processing:    20   45  25.1     45      85
Waiting:       20   41  25.3     40      85
Total:         29   52  24.4     55      95

Percentage of the requests served within a certain time (ms)
  50%     55     # 50%请求55ms内完成
  66%     57
  75%     57
  80%     90
  90%     95
  95%     95
  98%     95
  99%     95
 100%     95 (longest request)
```
上面看到10次请求，有6失败，查一下log
```
$ sudo tail /var/log/nginx/error.log 

2019/01/28 01:45:05 [error] 5951#0: *5 limiting requests, excess: 6.000 by zone "one", client: 127.0.0.1, server: localhost, request: "GET /phpinfo.php HTTP/1.0", host: "localhost"
2019/01/28 01:45:05 [error] 5951#0: *4 limiting requests, excess: 6.000 by zone "one", client: 127.0.0.1, server: localhost, request: "GET /phpinfo.php HTTP/1.0", host: "localhost"
2019/01/28 01:45:05 [error] 5951#0: *3 limiting requests, excess: 6.000 by zone "one", client: 127.0.0.1, server: localhost, request: "GET /phpinfo.php HTTP/1.0", host: "localhost"
2019/01/28 01:45:05 [error] 5951#0: *2 limiting requests, excess: 6.000 by zone "one", client: 127.0.0.1, server: localhost, request: "GET /phpinfo.php HTTP/1.0", host: "localhost"
```


