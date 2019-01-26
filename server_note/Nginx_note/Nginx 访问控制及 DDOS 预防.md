# Nginx 访问控制及 DDOS 预防
访问控制：</br>
一般网站的后台都不能让外部访问，所以要添加 IP 限制，通常只允许公司的 IP 访问。访问控制就是指只有符合条件的 IP 才能访问到这个网站的某个区域。</br>
</br>
DDOS 预防：</br>
DDOS 的特点是分布式，针对带宽和服务攻击，也就是四层流量攻击和七层应用攻击，相应的防御瓶颈四层在带宽，七层的多在架构的吞吐量。

## 访问控制
访问控制基于 ngx_http_access_module 模块，允许限制某些 IP 地址的客户端访问。</br>
ngx_http_access_module 只是很简单的访问控制，而在规则很多的情况下，使用 ngx_http_geo_module 模块变量更合适。</br>
详情： http://nginx.org/en/docs/http/ngx_http_geo_module.html
### 修改配置文件

## DDOS 防范
对于七层的应用攻击，我们还是可以做一些配置来防御的。</br>
使用 nginx 的 http_limit_conn 和 http_limit_req 模块通过限制连接数和请求数能相对有效的防御。</br>
* ngx_http_limit_conn_module 可以限制单个 IP 的连接数</br>
* ngx_http_limit_req_module 可以限制单个 IP 每秒请求数</br>

