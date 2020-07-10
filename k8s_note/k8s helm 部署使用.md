# helm 部署使用
## helm 部署
官网  https://github.com/helm/helm/releases
### 使用 shell 安装 helm
```
# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
# chmod 700 get_helm.sh
# ./get_helm.sh
```
### 编译包安装
下载二进制包
```
# wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
```
解压
```
tar -xvzf helm-v3.2.1-linux-amd64.tar.gz
mv ./linux-amd64 ../helm
```
设置软连
```
ln -s /sdata/usr/local/helm/helm /usr/bin/helm
```
查看 helm 版本
```
helm version
version.BuildInfo{Version:"v3.2.1", GitCommit:"fe51cd1e31e6a202cba7dead9552a6d418ded79a", GitTreeState:"clean", GoVersion:"go1.13.10"}
```
## 初始化 helm chart
添加 helm 镜像仓库
```
# helm repo add stable https://kubernetes-charts.storage.googleapis.com/
"stable" has been added to your repositories
```
更新镜像
```
helm repo update
```
搜索镜像
```
#  helm search repo mysql
NAME                                CHART VERSION    APP VERSION    DESCRIPTION                                       
stable/mysql                        1.6.3            5.7.28         Fast, reliable, scalable, and easy to use open-...
stable/mysqldump                    2.6.0            2.4.1          A Helm chart to help backup MySQL databases usi...
stable/prometheus-mysql-exporter    0.5.2            v0.11.0        A Helm chart for prometheus mysql exporter with...
stable/percona                      1.2.1            5.7.26         free, fully compatible, enhanced, open source d...
stable/percona-xtradb-cluster       1.0.3            5.7.19         free, fully compatible, enhanced, open source d...
stable/phpmyadmin                   4.3.5            5.0.1          DEPRECATED phpMyAdmin is an mysql administratio...
stable/gcloud-sqlproxy              0.6.1            1.11           DEPRECATED Google Cloud SQL Proxy                 
stable/mariadb                      7.3.14           10.3.22        DEPRECATED Fast, reliable, scalable, and easy t...
```
拉取 chart
```
# helm pull stable/mysql
# tar -xvzf ./mysql-1.6.3.tgz
# tree .
.
├── Chart.yaml
├── README.md
├── templates
│   ├── configurationFiles-configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── initializationFiles-configmap.yaml
│   ├── NOTES.txt
│   ├── pvc.yaml
│   ├── secrets.yaml
│   ├── serviceaccount.yaml
│   ├── servicemonitor.yaml
│   ├── svc.yaml
│   └── tests
│       ├── test-configmap.yaml
│       └── test.yaml
└── values.yaml


2 directories, 15 files
```
安装 mysql chart
```
# helm install stable/mysql --generate-name
NAME: mysql-1589334133
LAST DEPLOYED: Wed May 13 09:42:17 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-1589334133.default.svc.cluster.local


To get your root password run:


    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-1589334133 -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)


To connect to your database:


1. Run an Ubuntu pod that you can use as a client:


    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il


2. Install the mysql client:


    $ apt-get update && apt-get install mysql-client -y


3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql-1589334133 -p


To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306


    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql-1589334133 3306


    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```
查看 k8s pod
```
# kubectl get pod|grep mysql
mysql-1589334133-6d57d6768c-hc7rh   0/1     Running                 0          3m10s
```
使用 k8s secret 设置 mysql 认证信息
```
# MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-1589334133 -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
# echo $MYSQL_ROOT_PASSWORD
ssBiCxA5JP
```
查看 secret
```
# kubectl get secret mysql-1589334133 -o yaml
apiVersion: v1
data:
  mysql-password: R3g2WVhrTlhZMw==
  mysql-root-password: c3NCaUN4QTVKUA==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: mysql-1589334133
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2020-05-13T01:42:17Z"
  labels:
    app: mysql-1589334133
    app.kubernetes.io/managed-by: Helm
    chart: mysql-1.6.3
    heritage: Helm
    release: mysql-1589334133
  name: mysql-1589334133
  namespace: default
  resourceVersion: "3852206"
  selfLink: /api/v1/namespaces/default/secrets/mysql-1589334133
  uid: d117084d-b78a-4a77-bb68-2d6ee847a930
type: Opaque
```
关于 passwd 的 base64 转换
```
# echo -n 'ssBiCxA5JP' | base64
c3NCaUN4QTVKUA==

# echo 'c3NCaUN4QTVKUA==' | base64 --decode
ssBiCxA5JP
```
使用 k8s 拉取搭建一台 ubuntu 服务器用于测试连接数据库
```
kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
```
进入到 ubuntu 容器中安装 mysql-client
```
root@ubuntu:/# apt-get update && apt-get install mysql-client -y
```
映射pod端口 3306 到 k8s 外部
```
# kubectl port-forward svc/mysql-1589334133 3306
Forwarding from 127.0.0.1:3306 -> 3306
Forwarding from [::1]:3306 -> 3306

# netstat -tunpl|grep 3306
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      27433/kubectl       
tcp6       0      0 ::1:3306                :::*                    LISTEN      27433/kubectl    
```
于 k8s 外部访问 mysql
```
# mysql -h 127.0.0.1  -P3306 -u root -p
```


## helm chart 常用命令
查看 chart 信息
```
# helm show chart stable/mysql
# helm show all chart stable/mysql
```
查看 chart release 信息 
```
# helm list
NAME                NAMESPACE    REVISION    UPDATED                                    STATUS      CHART          APP VERSION
mysql-1589334133    default      1           2020-05-13 09:42:17.326429929 +0800 CST    deployed    mysql-1.6.3    5.7.28     
```
卸载 mysql
```
# helm uninstall mysql-1589334133 --keep-history
release "mysql-1589334133" uninstalled
```
查看历史状态，前提是有  --keep-history
```
helm status mysql-1589334133
```
只生成 yaml 不部署
```
#  helm install  stable/mysql --dry-run --debug --generate-name
install.go:159: [debug] Original chart version: ""
install.go:176: [debug] CHART PATH: /root/.cache/helm/repository/mysql-1.6.3.tgz


NAME: mysql-1590984246
LAST DEPLOYED: Mon Jun  1 12:04:10 2020
NAMESPACE: default
STATUS: pending-install
REVISION: 1
USER-SUPPLIED VALUES:
{}


COMPUTED VALUES:
affinity: {}
args: []
busybox:
  image: busybox
  tag: 1.29.3
configurationFiles: {}
configurationFilesPath: /etc/mysql/conf.d/
deploymentAnnotations: {}
extraInitContainers: |
  # - name: do-something
  #   image: busybox
  #   command: ['do', 'something']
extraVolumeMounts: |
  # - name: extras
  #   mountPath: /usr/share/extras
  #   readOnly: true
extraVolumes: |
  # - name: extras
  #   emptyDir: {}
image: mysql
imagePullPolicy: IfNotPresent
imageTag: 5.7.28
initContainer:
  resources:
    requests:
      cpu: 10m
      memory: 10Mi
initializationFiles: {}
livenessProbe:
  failureThreshold: 3
  initialDelaySeconds: 30
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 5
metrics:
  annotations: {}
  enabled: false
  flags: []
  image: prom/mysqld-exporter
  imagePullPolicy: IfNotPresent
  imageTag: v0.10.0
  livenessProbe:
    initialDelaySeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    initialDelaySeconds: 5
    timeoutSeconds: 1
  resources: {}
  serviceMonitor:
    additionalLabels: {}
    enabled: false
nodeSelector: {}
persistence:
  accessMode: ReadWriteOnce
  annotations: {}
  enabled: true
  size: 8Gi
podAnnotations: {}
podLabels: {}
readinessProbe:
  failureThreshold: 3
  initialDelaySeconds: 5
  periodSeconds: 10
  successThreshold: 1
  timeoutSeconds: 1
resources:
  requests:
    cpu: 100m
    memory: 256Mi
securityContext:
  enabled: false
  fsGroup: 999
  runAsUser: 999
service:
  annotations: {}
  port: 3306
  type: ClusterIP
serviceAccount:
  create: false
ssl:
  certificates: null
  enabled: false
  secret: mysql-ssl-certs
strategy:
  type: Recreate
testFramework:
  enabled: true
  image: dduportal/bats
  tag: 0.4.0
tolerations: []


HOOKS:
---
# Source: mysql/templates/tests/test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-1590984246-test
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    heritage: "Helm"
    release: "mysql-1590984246"
  annotations:
    "helm.sh/hook": test-success
spec:
  initContainers:
    - name: test-framework
      image: "dduportal/bats:0.4.0"
      command:
      - "bash"
      - "-c"
      - |
        set -ex
        # copy bats to tools dir
        cp -R /usr/local/libexec/ /tools/bats/
      volumeMounts:
      - mountPath: /tools
        name: tools
  containers:
    - name: mysql-1590984246-test
      image: "mysql:5.7.28"
      command: ["/tools/bats/bats", "-t", "/tests/run.sh"]
      volumeMounts:
      - mountPath: /tests
        name: tests
        readOnly: true
      - mountPath: /tools
        name: tools
  volumes:
  - name: tests
    configMap:
      name: mysql-1590984246-test
  - name: tools
    emptyDir: {}
  restartPolicy: Never
MANIFEST:
---
# Source: mysql/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-1590984246
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    release: "mysql-1590984246"
    heritage: "Helm"
type: Opaque
data:
  
  
  mysql-root-password: "MThYbGtjU0E0NQ=="
  
  
  
  
  mysql-password: "b2dGRktRRFVINw=="
---
# Source: mysql/templates/tests/test-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-1590984246-test
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    heritage: "Helm"
    release: "mysql-1590984246"
data:
  run.sh: |-
---
# Source: mysql/templates/pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-1590984246
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    release: "mysql-1590984246"
    heritage: "Helm"
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "8Gi"
---
# Source: mysql/templates/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-1590984246
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    release: "mysql-1590984246"
    heritage: "Helm"
  annotations:
spec:
  type: ClusterIP
  ports:
  - name: mysql
    port: 3306
    targetPort: mysql
  selector:
    app: mysql-1590984246
---
# Source: mysql/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-1590984246
  namespace: default
  labels:
    app: mysql-1590984246
    chart: "mysql-1.6.3"
    release: "mysql-1590984246"
    heritage: "Helm"


spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: mysql-1590984246
      release: mysql-1590984246
  template:
    metadata:
      labels:
        app: mysql-1590984246
        release: mysql-1590984246
    spec:
      serviceAccountName: default
      initContainers:
      - name: "remove-lost-found"
        image: "busybox:1.29.3"
        imagePullPolicy: "IfNotPresent"
        resources:
          requests:
            cpu: 10m
            memory: 10Mi
        command:  ["rm", "-fr", "/var/lib/mysql/lost+found"]
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
      # - name: do-something
      #   image: busybox
      #   command: ['do', 'something']
      
      containers:
      - name: mysql-1590984246
        image: "mysql:5.7.28"
        imagePullPolicy: "IfNotPresent"
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-1590984246
              key: mysql-root-password
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-1590984246
              key: mysql-password
              optional: true
        - name: MYSQL_USER
          value: ""
        - name: MYSQL_DATABASE
          value: ""
        ports:
        - name: mysql
          containerPort: 3306
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "mysqladmin ping -u root -p${MYSQL_ROOT_PASSWORD}"
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "mysqladmin ping -u root -p${MYSQL_ROOT_PASSWORD}"
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        # - name: extras
        #   mountPath: /usr/share/extras
        #   readOnly: true
        
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mysql-1590984246
      # - name: extras
      #   emptyDir: {}


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-1590984246.default.svc.cluster.local


To get your root password run:


    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql-1590984246 -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)


To connect to your database:


1. Run an Ubuntu pod that you can use as a client:


    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il


2. Install the mysql client:


    $ apt-get update && apt-get install mysql-client -y


3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql-1590984246 -p


To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306


    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql-1590984246 3306


    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```
