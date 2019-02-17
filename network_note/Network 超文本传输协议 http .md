# 超文本传输协议 http 原理及配置
超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。
所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。
## HTTP 原理
HTTP是一个客户端和服务器端请求和应答的标准（TCP）。客户端是终端用户，服务器端是网站。通过使用Web浏览器、网络爬虫或者其它的工具，客户端发起一个到服务器上指定端口（默认端口为80）的HTTP请求。应答的服务器上存储着（一些）资源，比如HTML文件和图像。</br>
通常，由HTTP客户端发起一个请求，建立一个到服务器指定端口（默认是80端口）的TCP连接。HTTP服务器则在那个端口监听客户端发送过来的请求。一旦收到请求，服务器（向客户端）发回一个状态行，比如"HTTP/1.1 200 OK"，和（响应的）消息，消息的消息体可能是请求的文件、错误消息、或者其它一些信息。</br>
通过HTTP或者HTTPS协议请求的资源由统一资源标示符（Uniform Resource Identifiers）（或者，更准确一些，URLs）来标识。</br>
详情可见 [Linux(CentOS7)-Nginx-MariaDB-PHP 架构 源码安装及配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/server_note/Nginx_note/Linux(CentOS7)-Nginx-MariaDB-PHP%20%E6%9E%B6%E6%9E%84%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE.MD)
## HTTP与DNS服务器
DNS 服务器可将域名和IP地址相互映射（url转ip）的一个分布式数据库，能够使人更方便地访问互联网。
## HTTP服务器与DNS服务器的配置
配置承接[动态主机配置协议 DHCP 原理及配置](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/network_note/Network%20%E5%8A%A8%E6%80%81%E4%B8%BB%E6%9C%BA%E9%85%8D%E7%BD%AE%E5%8D%8F%E8%AE%AE%20DHCP.md)中的拓扑和配置，
路由选择使用 OSPF，部分主机设置静态ip，部分主机与http服务器使用DHCP动态获取ip地址。</br>
</br>
本次配置的网络拓扑如下图所示：
</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/http_dns.png)
</br>
</br>
http server 的ip为 172.16.40.4，尝试在本机连接页面</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/http_server.png)</br>
</br>
在和http server 同一网段的pc 172.16.40.2 上尝试连接网页</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/http_pc.png)</br>
</br>
设置DNS服务器域名
```
url 为 cisco.com
http server ip 为 172.16.40.4
```
在 172.16.50.4 的pc上使用url尝试连接网页，可见 DNS的域名转换已成功</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Networking_note_images/http_url.png)</br>
