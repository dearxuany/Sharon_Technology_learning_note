# Nginx 报错 if 判断中设置带有 URI 代理
## 背景
Nginx 需根据真实来源 IP 来判断请求转发给哪个后端，当识别到为办公网调试IP时将请求转发给预生产环境，而其他外部请求转发给生产环境，且转发过程中需对接口 URI 做处理。</br>

调试期间出现如下报错：</br>
提示在 if 判断中，不支持使用带有 URI 的代理
```
#  nginx -t -c /sdata/etc/nginx/nginx.conf
nginx: [emerg] "proxy_pass" cannot have URI part in location given by regular expression, or inside named location, or inside "if" statement, or inside "limit_except" block in domain.com.cn_443.conf:210
nginx: configuration file /sdata/etc/nginx/nginx.conf test failed
```

## 解决办法
在 if 判断中，先将接口 rewrite 到目标接口，再做域名代理
```
  location /front/api/ai-coeus-api/ {
    proxy_read_timeout 180s;
    if ($remote_addr = 172.16.49.120){
        #rewrite /front/api/ai-coeus-api(/|$)(.*) /ai-coeus-api/$2 break;
        rewrite /front/api/ai-coeus-api/(.*) /ai-coeus-api/$1 break;
        proxy_pass http://uatinternal.domain.com;
    }
    proxy_pass http://qasinternal.domain.com/ai-coeus-api/;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
  }
```
以上 Nginx 配置包含如下步骤：
* https://web.domain.com/front/api/ai-coeus-api/ai-coeus-api/abc 重定向到 http://web.domain.com/ai-coeus-api/ai-coeus-api/abc
* 代理到  http://uatinternal.domain.com/ai-coeus-api/ai-coeus-api/abc 
* k8s ingress 将  /ai-coeus-api/ai-coeus-api/abc 转为 /ai-coeus-api/abc 给后端服务
