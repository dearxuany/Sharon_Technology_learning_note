# Nginx 504 Gateway Time-out 问题解决
浏览器访问页面发现接口报错
```
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx</center>
</body>
</html>
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
```
主要是由于后端服务响应慢导致的，但该接口为数据处理接口，暂不考虑优化，需再 nginx 层面调优。</br>
须添加以下参数，延长超时时间：
```
proxy_connect_timeout 300s;
proxy_send_timeout 300s;
proxy_read_timeout 300s;
```
整体配置:
```
server {
    listen 80;
    server_name {{ hostnames.audp }};
    access_log /root/share_logs/{{ hostnames.audp }}_access.log;
    error_log /root/share_logs/{{ hostnames.audp }}_error.log notice;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 8;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg   image/gif image/png application/json;
  
  location / {
    alias /sdata/www/dist/;
    try_files $uri $uri/ /index.html;
  }


  location /api/ {
    proxy_pass http://{{ hostnames.audp_api }}/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_connect_timeout 300s;
    proxy_send_timeout 300s;
    proxy_read_timeout 300s;
  }
}

```
