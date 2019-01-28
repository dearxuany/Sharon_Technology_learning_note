# Nginx 日志文件
```
[sunnylinux@centOSlearning logs]$ pwd
/usr/local/webserver/nginx/logs
[sunnylinux@centOSlearning logs]$ ls
access.log  error.log  nginx_error.log  nginx.pid
```
## access.log 连接日志
### 打开 access.log 
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
自动生成log文件
```
[sunnylinux@centOSlearning logs]$ ls
access.log  error.log  host.access.log  monitor.access.log  nginx_error.log  nginx.pid
```

### 查看 access.log
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

$ tail monitor.access.log
192.168.137.1 - - [29/Jan/2019:02:00:23 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
192.168.137.1 - - [29/Jan/2019:02:00:27 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36" "-"
```
