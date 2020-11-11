# k8s helm chart 报错 rendered manifests contain a resource that already exists

jenkins 调用 helm 部署应用到 k8s 失败，提示 missing key，服务在 k8s 内实例已 uninstall </br>
</br>
Error: rendered manifests contain a resource that already exists. Unable to continue with install: Service "ocr-server" in namespace "qas" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "ocr-server"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "qas"

```
# helm version
version.BuildInfo{Version:"v3.2.0", GitCommit:"e11b7ce3b12db2941e90399e874513fbd24bcb71", GitTreeState:"clean", GoVersion:"go1.13.10"}
```
解决办法参考文档 </br>
https://github.com/helm/helm-2to3/issues/147 </br>
https://github.com/helm/helm/pull/7649 </br>
</br>
此处主要是上一次应用部署失败了，service 已部署完毕但是 pod 没有成功启动，应用被回滚时没有同时去除已新增的 service。在新的一轮部署时，由于 helm 在应用的部署逻辑上没有识别到上个版本的 service 而导致两个应用版本冲突，新版本无法正常部署。短期解决办法是将已有的错误 service 直接从 k8s 里删除，长期需优化 helm chart，添加 release metadata 和 managed-by label。
```
> kubectl get all -n qas |grep ocr
pod/ocr-display-front-69bf8d9f4-qwltt                          1/1     Running   0          4d9h
service/ocr-server                         ClusterIP   172.21.1.175    <none>        8080/TCP                     15h
service/ocr-display-front                          ClusterIP   172.21.6.207    <none>        80/TCP                       70d
deployment.apps/ocr-display-front                          1/1     1            1           70d
replicaset.apps/ocr-display-front-5c75df85b6                          0         0         0       32d
replicaset.apps/ocr-display-front-67fb89595f                          0         0         0       20d
replicaset.apps/ocr-display-front-69bf8d9f4                           1         1         1       5d18h
replicaset.apps/ocr-display-front-6bd9f6cfc5                          0         0         0       70d
replicaset.apps/ocr-display-front-dcb844f96                           0         0         0       32d


> kubectl describe service ocr-server  -n qas
Name:              ocr-server
Namespace:         qas
Labels:            cattle.io/creator=norman
Annotations:       field.cattle.io/targetWorkloadIds: 
Selector:          <none>
Type:              ClusterIP
IP:                172.21.1.175
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
删除无法适配的旧资源即可重新部署新版本
```
> kubectl get services -n qas|grep ocr
ocr-server                         ClusterIP   172.21.1.175    <none>        8080/TCP                     16h
ocr-display-front                          ClusterIP   172.21.6.207    <none>        80/TCP                       70d

> kubectl delete service ocr-server -n qas
service "ocr-server" deleted
```
helm 正常部署的 service 描述会带有 helm 相关的 Labels 和 Annotations</br>
Labels </br>
* app.kubernetes.io/managed-by=Helm </br>

Annotations </br>
* meta.helm.sh/release-name: <RELEASE_NAME>
* meta.helm.sh/release-namespace: <RELEASE_NAMESPACE>
```
> kubectl describe service ocr-server -n qas
Name:              ocr-server
Namespace:         qas
Labels:            app=ocr-server
                   app.kubernetes.io/managed-by=Helm
                   release=ocr-server
Annotations:       meta.helm.sh/release-name: ocr-server
                   meta.helm.sh/release-namespace: qas
Selector:          app=ocr-server
Type:              ClusterIP
IP:                172.21.13.234
Port:              ocr-server  8080/TCP
TargetPort:        5000/TCP
Endpoints:         172.18.117.19:5000
Session Affinity:  None
Events:            <none>
```

