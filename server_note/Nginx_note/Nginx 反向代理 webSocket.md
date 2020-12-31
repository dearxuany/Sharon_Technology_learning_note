# Nginx 反向代理 webSocket
## 背景
WebSockets的应用程序会在客户端和服务端保持一个长时间工作的连接，用来将连接从HTTP升级到WebSocket的HTTP升级机制使用HTTP的Upgrade和Connection协议头。</br>
Websoket是一个持久化的协议，相对于HTTP这种非持久的协议来说，当服务器完成协议升级后（HTTP->Websocket），服务端就可以主动推送信息给客户端。 只需要经过一次HTTP请求，就可以做到源源不断的信息传送。</br>
Websocket只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，也就避免了HTTP的非状态性，服务端会一直知道你的信息，直到关闭请求，这样就解决了接线员要反复解析HTTP协议，还要查看identity info的信息。</br>
</br>
WebSocket协议的应用场景:
* 即时聊天通信
* 多玩家游戏
* 在线协同编辑/编辑
* 实时数据流的拉取与推送
* 体育/游戏实况
* 实时地图位置
## 配置
Nginx webSocket 配置
```
  location /open-api/webSocketServer {
    proxy_pass http://domain.com/open-api/webSocketServer;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
```
webSocket 相关配置
* proxy_set_header Upgrade 把代理时http请求头的Upgrade 设置为原来http请求的请求头，wss协议的请求头为websocket</br>
* proxy_set_header Connection 因为代理wss协议，所以http请求头的Connection设置为Upgrade</br>
* proxy_http_version 设置代理使用的HTTP协议版本，默认使用的版本是1.0，而1.1版本则推荐在使用keepalive连接时一起使用，当没有使用http1.1的时候，后端服务会返回101错误，然后断开连接</br>
</br>
参考文档 https://www.cnblogs.com/ybyqjzl/p/10350732.html


