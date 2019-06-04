# Nginx 日志显示真实IP
## 查看 nginx 安装模块
获得真实IP需要安装 --with-http_realip_module
```
$ 2>&1 nginx -V | tr ' '  '\n'
nginx
version:
nginx/1.12.2
built
by
gcc
4.8.5
20150623
(Red
Hat
4.8.5-16)
(GCC)

built
with
OpenSSL
1.0.2k-fips

26
Jan
2017
TLS
SNI
support
enabled
configure
arguments:
--prefix=/usr/share/nginx
--sbin-path=/usr/sbin/nginx
--modules-path=/usr/lib64/nginx/modules
--conf-path=/etc/nginx/nginx.conf
--error-log-path=/var/log/nginx/error.log
--http-log-path=/var/log/nginx/access.log
--http-client-body-temp-path=/var/lib/nginx/tmp/client_body
--http-proxy-temp-path=/var/lib/nginx/tmp/proxy
--http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi
--http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi
--http-scgi-temp-path=/var/lib/nginx/tmp/scgi
--pid-path=/run/nginx.pid
--lock-path=/run/lock/subsys/nginx
--user=nginx
--group=nginx
--with-file-aio
--with-ipv6
--with-http_auth_request_module
--with-http_ssl_module
--with-http_v2_module
--with-http_realip_module
--with-http_addition_module
--with-http_xslt_module=dynamic
--with-http_image_filter_module=dynamic
--with-http_geoip_module=dynamic
--with-http_sub_module
--with-http_dav_module
--with-http_flv_module
--with-http_mp4_module
--with-http_gunzip_module
--with-http_gzip_static_module
--with-http_random_index_module
--with-http_secure_link_module
--with-http_degradation_module
--with-http_slice_module
--with-http_stub_status_module
--with-http_perl_module=dynamic
--with-mail=dynamic
--with-mail_ssl_module
--with-pcre
--with-pcre-jit
--with-stream=dynamic
--with-stream_ssl_module
--with-google_perftools_module
--with-debug
--with-cc-opt='-O2
-g
-pipe
-Wall
-Wp,-D_FORTIFY_SOURCE=2
-fexceptions
-fstack-protector-strong
--param=ssp-buffer-size=4
-grecord-gcc-switches
-specs=/usr/lib/rpm/redhat/redhat-hardened-cc1
-m64
-mtune=generic'
--with-ld-opt='-Wl,-z,relro
-specs=/usr/lib/rpm/redhat/redhat-hardened-ld
-Wl,-E'
```

## 修改 nginx 配置
打开 default.conf配置文件，在 location / {}中添加以下内容：其中，ip_range1，2，...，x 指WAF的回源IP地址，需要分多条分别添加。
```
set_real_ip_from ip_range1;
set_real_ip_from ip_range2;
...
set_real_ip_from ip_rangex;
real_ip_header X-Forwarded-For;
```
添加 waf 源IP地址范围配置
```
vim get_real_ip_from.conf
```
在 location / {} 中include get_real_ip_from.conf
```
    location / {
            proxy_pass http://172.18.65.46:5601;
            #proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;

            include get_real_ip_from.conf;
        }

```

## 修改nginx日志格式
log_format一般在 nginx.conf配置文件中的http配置部分。在 log_format中，添加 x-forwarded-for字段，替换原来remote-address字段
```
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent $request_time  "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```
