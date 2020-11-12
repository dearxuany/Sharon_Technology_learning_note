# k8s nginx ingress 开启跨域访问
浏览器访问报错
```
Access to fetch at 'https://gateway.domainname.com/shortlink-service/short-link' from origin 'https://query.domainname.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.
```
解决方案
```
Access-Control-Allow-Origin 字段必须指定域名，不能为*，如果带了凭据，在需要传Cookie等时，服务端的Access-Control-Allow-Origin必须配置具体的具体的域名
Access-Control-Allow-Credentials 必须设置为 true，让Vue项目在跨域时允许请求发送凭据，如cookie、HTTP认证及客户端SSL证明
```
k8s nginx ingress 开启跨域访问，需在 annotations 中添加以下几项配置：
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: external-ingress   # 绑定外网访问 IP
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"   
    nginx.ingress.kubernetes.io/cors-allow-headers: '*'
    nginx.ingress.kubernetes.io/cors-allow-methods: '*'
    nginx.ingress.kubernetes.io/cors-allow-origin: https://query.domainname.com  
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /short-link/$2 # 接口跳转
```
参考文章
* Vue 加入 withCredentials 后无法进行跨域请求 https://blog.csdn.net/liyuling52011/article/details/80013725
* k8s ingress 增加跨域配置 https://www.cnblogs.com/lixinliang/p/13521787.html


