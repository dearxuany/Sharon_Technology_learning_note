# k8s ImagePullSecrets 设置使用阿里云私有镜像仓库
## 背景
创建 deployment 显示 ImagePullBackOff 无法拉取镜像   
```
NAMESPACE            NAME                                            READY   STATUS              RESTARTS   AGE
dev         coes-test-58d75cf854-fkl76                   0/1     ImagePullBackOff    0          96m
```
选择新建密钥需要配置以下参数
* 密钥名称（regsecret）： 指定密钥的键名称，可自行定义。
* 仓库域名（docker-server）：指定 Docker 仓库地址。若输入阿里云容器服务镜像仓库，会默认补齐。
* 用户名（docker-username）： 指定 Docker 仓库用户名。若使用阿里云容器镜像服务，则为阿里云登录账号（包括阿里云主账号和子账号）。
* 密码（docker-password）：指定 Docker 仓库登录密码。若使用阿里云容器镜像服务，即容器registry独立登录密码。
* 邮箱（docker-email）：指定邮件地址。非必选。


## k8s ImagePullSecrets 设置
创建 secret
```
kubectl --namespace dev create secret docker-registry aliyun-docker-registry --docker-server=registry.cn-shenzhen.aliyuncs.com --docker-username=ops_acr@1060896234 --docker-password=passwd --docker-email=DOCKER_EMAIL
secret/aliyun-docker-registry created
```
查看 secret
```
# kubectl get secret aliyun-docker-registry
NAME                     TYPE                             DATA   AGE
aliyun-docker-registry   kubernetes.io/dockerconfigjson   1      2m21s
```
创建时如果不设置 namespace 则创建在 default，不同 namespaces 可以有同名 secrets
```
# kubectl  --namespace dev get secret aliyun-docker-registry -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJyZWdpc3RyeS55hbWUiOiJvcHNfYWNyQDEwNjA4OTYyMzQzMzQyODQiLCJwYXNzd29yZCIXRoIjoiYjNCelg1qZzBPbkJVTlc5dVFtRjBZM2hsUmtac1RHcz0ifX19
kind: Secret
metadata:
  creationTimestamp: "2020-06-01T11:12:09Z"
  name: aliyun-docker-registry
  namespace: dev
  resourceVersion: "76896"
  selfLink: /api/v1/namespaces/dev/secrets/aliyun-docker-registry
  uid: 924548d8-722d-409a-86c0-9523ced84bdb
type: kubernetes.io/dockerconfigjson
```
转换标注输出
```
# kubectl get secret aliyun-docker-registry  --output="jsonpath={.data.\.dockerconfigjson}" | base64 -d
{"auths":{"registry.cn-shenzhen.aliyuncs.com":{"username":"ops_acr@106089","password":"passwd","auth":"b3BzX2FjckAxMDYwODk2MjGs="}}}
```
在 yaml 管理文件中使用 ImagePullSecrets
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coes-test
  labels:
    app: coes-test
    release: coes-test
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: coes-test
  template:
    metadata:
      labels:
        app: coes-test
    spec:
      imagePullSecrets:
      - name: aliyun-docker-registry
      containers:
        - name: coeus-api
          image: registry.cn-shenzhen.aliyuncs.com/namespace/coeus:build_26
          imagePullPolicy: IfNotPresent
          ports:
            - name: port
              containerPort: 5005
              protocol: TCP
          command: ["/bin/bash"]
          args: ["-c","/root/app/start_rasa.sh"]
          readinessProbe:
            failureThreshold: 3
            tcpSocket:
              port: 5005
```
