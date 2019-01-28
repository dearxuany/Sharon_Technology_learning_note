# Nginx 日志文件与切分备份
```
[sunnylinux@centOSlearning logs]$ pwd
/usr/local/webserver/nginx/logs
[sunnylinux@centOSlearning logs]$ ls
access.log  error.log  nginx_error.log  nginx.pid
```
## access.log 连接日志
### 配置 access.log 
修改 nginx.conf
```

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
```
删除注释，logs/access.log 在默认路径下logs目录中生成access.log，格式为main所定义，也即是上面的log_format部分
```
 server {
        listen  1234;
        server_name  localhost;
        access_log  logs/monitor.access.log  main;   # 自定义log

        location / {
            root  html;
            index  nginx_first.html; }
    }

```
自动生成log文件，log删掉之后，要reload才能再次生成
```
[sunnylinux@centOSlearning logs]$ ls
access.log  error.log  host.access.log  monitor.access.log  nginx_error.log  nginx.pid
```

### 查看 access.log
log 如果不被删除，会一直使用同一个文件记录
```
$ tail access.log
192.168.137.1 - - [29/Jan/2019:01:08:14 +0800] "GET / HTTP/1.1" 403 570 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:08:14 +0800] "GET /favicon.ico HTTP/1.1" 403 570 "http://192.168.137.100:1234/" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:08:14 +0800] "GET / HTTP/1.1" 403 570 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:08:14 +0800] "GET /favicon.ico HTTP/1.1" 403 570 "http://192.168.137.100:1234/" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:42 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:43 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:44 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:46 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:52 +0800] "GET / HTTP/1.1" 200 1035 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
192.168.137.1 - - [29/Jan/2019:01:09:53 +0800] "GET /favicon.ico HTTP/1.1" 404 570 "http://192.168.137.100:1234/" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
```
tail -n 查看最后5行，-f 循环查看动态显示文件内容，即是浏览器访问该页即可看到访问记录
```
$ tail -n 5 -f  monitor.access.log
192.168.137.1 - - [29/Jan/2019:02:00:23 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
192.168.137.1 - - [29/Jan/2019:02:00:27 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
192.168.137.1 - - [29/Jan/2019:02:07:36 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
192.168.137.1 - - [29/Jan/2019:02:21:00 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
192.168.137.1 - - [29/Jan/2019:02:21:05 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
```
注意浏览器的缓存问题，有缓存不会生成新的请求

## 日志拆分备份
请求访问太大的时候，为避免产生的log文件过大或丢失，要把日志进行切分备份，一般一天一备份。<br>
script 配合 linux 定时任务，备份并记录时间，生成新的log文件。<br>
过程：找日志文件-设日期和目的路径-关掉nginx-移动并改名日志文件到目的路径-重启nginx并自动生成新的日志文件<br>
[Python3 实现 nginx 日志定时切分与备份](https://github.com/dearxuany/python_program/tree/master/Nginx_log_backup)

## 日志分析相关工具
 ELKR ( = Elasticsearch + Logstash + Kibana + Redis ) 一套完整的 Nginx 日志分析技术栈 https://www.elastic.co/ 
 
 

