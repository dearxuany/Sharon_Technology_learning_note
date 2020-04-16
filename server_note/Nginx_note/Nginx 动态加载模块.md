# Nginx 动态加载模块

## 查看nginx详细信息
```
# nginx -V
nginx version: nginx/1.16.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/sdata/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module
```
## 动态加自带模块
### 编译
```
wget http://nginx.org/download/nginx-1.16.0.tar.gz
tar -zxf nginx-1.16.0.tar.gz
cd nginx-1.16.0
```
增加with-http_realip_module新模块，增加 --with-http_realip_module 参数
```
./configure --prefix=/sdata/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module
make
```
注意：不要make install，还需要将之前安装Nginx的参数全部加上

### 替换
```
mv /sdata/usr/local/nginx/sbin/nginx /sdata/usr/local/nginx/sbin/nginx.bak
cp ./objs/nginx /sdata/usr/local/nginx/sbin/
```
### 检查
```
# nginx -V
nginx version: nginx/1.16.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/sdata/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module
```
## 动态加第三方模块
### 获取echo-nginx-module
只解压，不编译，不删除
```
cd /sdata/usr/local/src/
wget https://github.com/openresty/echo-nginx-module/archive/v0.61.tar.gz
tar -zxvf v0.61.tar.gz
```

### 获取nginx
```
wget http://nginx.org/download/nginx-1.16.0.tar.gz
tar -zxf nginx-1.16.0.tar.gz
cd nginx-1.16.01.3.3 
```
### 增加echo-nginx-module新模块
增加参数 --add-module=/sdata/usr/local/src/echo-nginx-module-0.61

```
./configure --prefix=/sdata/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module --with-http_realip_module --add-module=/sdata/usr/local/src/echo-nginx-module-0.61
make
```
注意：不要make install

### 替换
```
mv /sdata/usr/local/nginx/sbin/nginx /sdata/usr/local/nginx/sbin/nginx.bak
cp ./objs/nginx /sdata/usr/local/nginx/sbin/
```
### 检查
```
# nginx -V
```
