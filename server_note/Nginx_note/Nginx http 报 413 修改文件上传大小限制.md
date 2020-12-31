# Nginx http 报 413 修改文件上传大小限制
页面上传文件接口报 413
```
xhr.js:184 POST https://domain.com/open-api/medical_report 500
(anonymous) @ xhr.js:184
e.exports @ xhr.js:13
e.exports @ dispatchRequest.js:52
Promise.then (async)
u.request @ Axios.js:61
(anonymous) @ bind.js:9
u @ request.js:45
analysisImage @ ImageAnalysis.vue:145
nt @ vue.runtime.esm.js:1854
n @ vue.runtime.esm.js:2179
nt @ vue.runtime.esm.js:1854
En.e.$emit @ vue.runtime.esm.js:3888
handleClick @ element-ui.common.js:9417
nt @ vue.runtime.esm.js:1854
n @ vue.runtime.esm.js:2179
Qr.o._wrapper @ vue.runtime.esm.js:6917
ImageAnalysis.vue:156 Error: Request failed with status code 500
    at e.exports (createError.js:16)
    at e.exports (settle.js:17)
    at XMLHttpRequest.d.onreadystatechange (xhr.js:69)
```
Nginx 限制了上传文件大小，修改nginx配置文件，配置客户端请求大小和缓存大小
```
    client_max_body_size 10M;
    client_body_buffer_size 10M;
```
client_max_body_size 默认 1M，表示 客户端请求服务器最大允许大小，在“Content-Length”请求头中指定。如果请求的正文数据大于client_max_body_size，HTTP协议会报错 413 Request Entity Too Large。
传输的数据大于client_max_body_size，一定是传不成功的。小于client_body_buffer_size直接在内存中高效存储。如果大于client_body_buffer_size小于client_max_body_size会存储临时文件，临时文件一定要有权限，client_body_temp 可指定路径，默认该路径值是/tmp。
如果追求效率，就设置 client_max_body_size client_body_buffer_size相同的值，这样就不会存储临时文件，直接存储在内存了。

```
# 完整配置

  server {
    listen 80;
    server_name {{ hostnames.filter }};
    access_log /root/logs/{{ hostnames.filter }}_access.log;
    error_log /root/logs/{{ hostnames.filter }}_error.log notice;
    
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 8;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg   image/gif image/png application/json;
    client_max_body_size 10M;
    client_body_buffer_size 10M;
  
  location / {
    alias /sdata/www/dist/;
    index index.html;
    try_files $uri $uri/ /index.html;
  }
  
  location /api {
    proxy_pass http://{{ hostnames.filter_api }};
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```
