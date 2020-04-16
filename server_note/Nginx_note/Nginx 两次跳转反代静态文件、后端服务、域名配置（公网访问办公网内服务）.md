# Nginx 两次跳转反代静态文件、后端服务、域名配置（公网访问办公网内服务）

## 背景

* 办公网入网 nat 只开放 443 指向 10.0.0.2 上的 nginx，此机器为入网 nginx ，部署较多服务，禁止除运维以外人员登陆。
* 开发环境的静态文件需要使用 nginx 做代理，使用阿里云上备案的域名，使用阿里云上的云解析服务来将此域名指向我司办公环境的入网 ip。
* nginx 无法代理位于不同服务器上的静态文件，故需要两个 nginx ，一个将入网请求反代到后端服务器，一个在开发服务器上用于代理静态文件。
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Nginx_images/nginx%20%E4%B8%A4%E6%AC%A1%E8%B7%B3%E8%BD%AC%E5%8F%8D%E4%BB%A3%E9%9D%99%E6%80%81%E6%96%87%E4%BB%B6%E3%80%81%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%E3%80%81%E5%9F%9F%E5%90%8D%E9%85%8D%E7%BD%AE%EF%BC%88%E5%85%AC%E7%BD%91%E8%AE%BF%E9%97%AE%E5%8A%9E%E5%85%AC%E7%BD%91%E5%86%85%E6%9C%8D%E5%8A%A1%EF%BC%89.png?raw=true)


## 具体配置
入网 nginx
```
server {
    listen 80;
    server_name ocrlabel.domainname.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}


server {
    #listen 80;
    listen 443 ssl;
    server_name ocrlabel.domainname.com;




    access_log /data2/log/nginx/ocrlabel.domainname.com_access.log;
    error_log /data2/log/nginx/ocrlabel.domainname.com_error.log notice;




    ssl_protocols       SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_certificate     cert/318081__domainname.com.pem;
    ssl_certificate_key cert/318081__domainname.com.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;


    location / {
            proxy_pass http://10.0.0.160:80;
            #proxy_http_version 1.1;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_connect_timeout 600;
            proxy_read_timeout 600;
            proxy_send_timeout 600;
            proxy_cache_bypass $http_upgrade;
        }
}
```

后端 nginx
```
# cat ocrlabel.domainname.com.conf
server {
    listen 80;
    server_name 10.0.0.160;

    access_log /sdata/var/log/nginx/ocrlabel.domainname.com_access.log main;
    error_log /sdata/var/log/nginx/ocrlabel.domainname.com_error.log notice;

    add_header Content-Security-Policy upgrade-insecure-requests;

    root /var/www/html/LabelMeAnnotationTool/;

    location / {
        if ( $request_uri = "/" ){
            rewrite "/" https://ocrlabel.domainname.com/LabelMeAnnotationTool/tool.html break;
        }
        proxy_pass http://10.0.0.160:6000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $remote_addr;


    }
}
```
