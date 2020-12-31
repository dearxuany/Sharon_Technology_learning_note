# Nginx 调整文件下载大小限制
nginx代理nginx时，前端用户请求下载文件， nginx代理会先从后端nginx拿到文件并缓存到本地，然后响应给客户端。</br>
</br>
Nginx 需添加以下配置：
```
proxy_request_buffering off; //禁用请求缓冲（不加的话大文件上传可能会在前端报415）
proxy_buffering off; //禁用缓冲（不加的话大文件上传可能会在前端报415）
```
完整配置
```
  location /knowledgeApi/ {
      proxy_pass http://backend:8786/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $remote_addr;
      proxy_request_buffering off;
      proxy_buffering off;
      #proxy_send_timeout 90;
      #proxy_read_timeout 90;
  }
```
