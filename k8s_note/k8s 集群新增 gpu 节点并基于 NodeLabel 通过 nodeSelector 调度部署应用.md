# k8s 集群新增 gpu 节点并基于 NodeLabel 通过 nodeSelector 调度部署应用
## 准备
### kubernetes 集群安装 gpu 设备插件
查看 kubernetes 版本 </br>
参考文档： https://help.aliyun.com/document_detail/86496.html?spm=a2c4g.11186623.6.640.160029a72x0EFR
```
> kubectl versionClient Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.8", GitCommit:"ec6eb119b81be488b030e849b9e64fda4caaf33c", GitTreeState:"clean", BuildDate:"2020-03-12T21:00:06Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}Server Version: version.Info{Major:"1", Minor:"16+", GitVersion:"v1.16.9-aliyun.1", GitCommit:"4f7ea78", GitTreeState:"", BuildDate:"2020-05-08T07:29:59Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

### 将 gpu 节点新加到 kubernetes 集群
由于现在使用的阿里云 ecs gpu 实例规格无法支持阿里自研的 k8s 网络组件 Terway，故还是需要更换实例规格。</br>
参考文档： https://www.alibabacloud.com/help/zh/doc-detail/97467.html </br>
</br>
由于需要新购置 gpu 实例故直接选择扩容集群添加新的 gpu worker 节点。</br>
考虑到新增的节点尚未部署完善，为防止集群已有 pod 被调度到 gpu 节点，需先将该新建 gpu 节点设置为不可调度。</br>
参考文档： https://help.aliyun.com/document_detail/87092.html?spm=a2c4g.11186623.6.655.20657000PK0GXt
```
kubectl cordon node-name
```
如原来节点上有 pod 需要将该节点上的 pod 移到其他节点
```
kubectl drain node-name --grace-period=120 --ignore-daemonsets=true
```
查看新增节点 gpu 驱动版本
```
[root@alihn1-qas-k8s-w31212 ~]# nvidia-smi
Thu Jul 23 13:48:10 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.87.01    Driver Version: 418.87.01    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla P4            On   | 00000000:00:07.0 Off |                    0 |
| N/A   33C    P8     7W /  75W |      0MiB /  7611MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
确认 nvdia-docker 配置为 runtime
```
# cat /etc/docker/daemon.json
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "10"
    },
    "bip": "169.254.123.1/24",
    "oom-score-adjust": -1000,
    "registry-mirrors": ["https://pqbap4ya.mirror.aliyuncs.com"],
    "storage-driver": "overlay2",
    "storage-opts":["overlay2.override_kernel_check=true"],
    "live-restore": true
}
```
存储 nas 挂载到 k8s 新增节点上
```
yum install nfs-utils
mkdir -p /share_log_mount
mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 28faebv72.cn-shenzhen.nas.aliyuncs.com:/ /share_log_mount
echo  "28fav72.cn-shenzhen.nas.aliyuncs.com:/ /share_log_mount nfs vers=4,minorversion=0,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev,noresvport 0 0" | sudo tee -a /etc/fstab
```
节点维护完毕后重新开启调度（注意先修改）




## Kubernetes GPU 节点调度
基于 NodeLabel 来进行 kubernetes gpu 节点调度

### 查看节点标签
由于是阿里云直接于 k8s 扩容出来的机器，故已有GPU节点的标签，如果是把已有机器添加到集群则需要自行配置
```
>  kubectl get nodes
NAME                          STATUS                     ROLES    AGE   VERSION
alihn1-qas-k8s-w21249         Ready                      <none>   75d   v1.16.9-aliyun.1
alihn1-qas-k8s-w31201         Ready                      <none>   75d   v1.16.9-aliyun.1
alihn1-qas-k8s-w31207         Ready                      <none>   37d   v1.16.9-aliyun.1
alihn1-qas-k8s-w31212         Ready,SchedulingDisabled   <none>   23h   v1.16.9-aliyun.1


>  kubectl describe node alihn1-qas-k8s-w31212
Name:               alihn1-qas-k8s-w31212
Roles:              <none>
Labels:             ack.aliyun.com=c366b4ab1094e4e85bcb4c3dd64fa32c6
                    alibabacloud.com/nodepool-id=np496f712af36a449db6e3e148a9308190
                    aliyun.accelerator/nvidia_count=1   # GPU核心数量
                    aliyun.accelerator/nvidia_mem=7611MiB  # GPU显存，单位为MiB
                    aliyun.accelerator/nvidia_name=Tesla-P4  # nvida设备的GPU计算卡名称
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=ecs.gn5i-c4g1.xlarge
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=cn-shenzhen
                    failure-domain.beta.kubernetes.io/zone=cn-shenzhen-d
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=alihn1-qas-k8s-w31212.snail
                    kubernetes.io/os=linux
                    topology.diskplugin.csi.alibabacloud.com/zone=cn-shenzhen-d
Annotations:        csi.volume.kubernetes.io/nodeid:
                      {"diskplugin.csi.alibabacloud.com":"i-wz9fxcenyic1zgdhhw7s","nasplugin.csi.alibabacloud.com":"i-wz9fxcenyic1zgdhhw7s","ossplugin.csi.aliba...
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 22 Jul 2020 06:48:19 +0000
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  KernelDeadlock       False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:01:53 +0000   KernelHasNoDeadlock          kernel has no deadlock
  ReadonlyFilesystem   False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:01:53 +0000   FilesystemIsReadOnly         Filesystem is read-only
  FDPressure           False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:01:54 +0000   NodeHasNoFDPressure          node has no fd pressure
  RAMRoleError         False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:01:52 +0000   NodeHasRAMRole               node has ram role
  NTPProblem           False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:01:53 +0000   NTPIsUp                      ntp service is up
  NvidiaXidError       False   Thu, 23 Jul 2020 06:44:22 +0000   Thu, 23 Jul 2020 03:03:53 +0000   NodeHasNoNvidiaXidError      Node has no Nvidia Xid error occured
  MemoryPressure       False   Thu, 23 Jul 2020 06:43:39 +0000   Wed, 22 Jul 2020 06:48:19 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Thu, 23 Jul 2020 06:43:39 +0000   Wed, 22 Jul 2020 06:48:19 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Thu, 23 Jul 2020 06:43:39 +0000   Wed, 22 Jul 2020 06:48:19 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Thu, 23 Jul 2020 06:43:39 +0000   Wed, 22 Jul 2020 06:48:45 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.18.31.212
  Hostname:    alihn1-qas-k8s-w31212
Capacity:
cpu:                4   # cpu 核心数
ephemeral-storage:  82437788Ki
hugepages-1Gi:      0
hugepages-2Mi:      0
memory:             16266252Ki
nvidia.com/gpu:     1   # gpu 核心数
pods:               23  # pods 限制数
Allocatable:
cpu:                4
ephemeral-storage:  75974665296
hugepages-1Gi:      0
hugepages-2Mi:      0
memory:             15242252Ki
nvidia.com/gpu:     1
pods:               23
System Info:
Machine ID:                 20200426154603174201708213343640
System UUID:                1CB97BA8-40E3-408F-B547-4E6FC96BAEC4
Boot ID:                    ad685e09-414c-4cb0-813c-6d8707915d93
Kernel Version:             3.10.0-1062.18.1.el7.x86_64
OS Image:                   CentOS Linux 7 (Core)
Operating System:           linux
Architecture:               amd64
Container Runtime Version:  docker://19.3.5
Kubelet Version:            v1.16.9-aliyun.1
Kube-Proxy Version:         v1.16.9-aliyun.1
ProviderID:                  cn-shenzhen.i-wz9fxcenyic1zgdhhw7s
Non-terminated Pods:         (7 in total)
  Namespace                  Name                                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                                ------------  ----------  ---------------  -------------  ---
  cattle-system              cattle-node-agent-p7vz6                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         23h
  kube-system                ack-node-problem-detector-daemonset-m9qbj           100m (2%)     100m (2%)   200Mi (1%)       200Mi (1%)     23h
  kube-system                csi-plugin-tcrlq                                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         23h
  kube-system                kube-proxy-worker-m4l2w                             0 (0%)        0 (0%)      0 (0%)           0 (0%)         23h
  kube-system                logtail-ds-7sjgd                                    100m (2%)     500m (12%)  256Mi (1%)       1Gi (6%)       23h
  kube-system                nvidia-device-plugin-alihn1-qas-k8s-w31212          500m (12%)    500m (12%)  300Mi (2%)       300Mi (2%)     23h
  kube-system                terway-eniip-vr25q                                  350m (8%)     350m (8%)   100Mi (0%)       256Mi (1%)     23h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                1050m (26%)  1450m (36%)
  memory             856Mi (5%)   1780Mi (11%)
  ephemeral-storage  0 (0%)       0 (0%)
  nvidia.com/gpu     0            0
Events:              <none>
```
 同一类型的GPU云服务器的GPU计算卡名称相同，可通过该标签筛选节点
```
>  kubectl get nodes -l aliyun.accelerator/nvidia_name=Tesla-P4
NAME                          STATUS                     ROLES    AGE   VERSION
alihn1-qas-k8s-w31212         Ready,SchedulingDisabled   <none>   24h   v1.16.9-aliyun.1
```
### yaml 通过 nodeSelector 部署应用
#### 选择 gpu 节点
使用 nodeSelector 通过 Labels 指定使用 gpu 的项目 pod 调度到 gpu 节点
```
# 简单样例

# Define the tensorflow deployment
apiVersion: apps/v1
kind: Deploymentmetadata:
  name: tf-notebook
  labels:
    app: tf-notebookspec:
  replicas: 1
  selector: # define how the deployment finds the pods it mangages
    matchLabels:
      app: tf-notebook
  template: # define the pods specifications
    metadata:
      labels:
        app: tf-notebook
    spec:
      nodeSelector:                                                  #注意
        aliyun.accelerator/nvidia_name: Tesla-M40
      containers:
      - name: tf-notebook
        image: tensorflow/tensorflow:1.4.1-gpu-py3
        resources:
          limits:
            nvidia.com/gpu: 1                                        #注意
        ports:
        - containerPort: 8888
          hostPort: 8888
        env:
          - name: PASSWORD
            value: mypassw0rdv
```

#### 排除 gpu 节点
通过节点亲和性排除含有 nvidia_name Labels key 的节点，防止不使用 gpu 的项目 pod 被调度到 gpu 节点
```
# 简单样例

apiVersion: v1
kind: Pod
metadata:
  name: not-in-gpu-node
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: aliyun.accelerator/nvidia_name
            operator: DoesNotExist
  containers:
  - name: not-in-gpu-node
    image: nginx
```
#### helm chart 生成 k8s 部署 yaml
helm chart tree
```
# tree -L 2
.
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml


1 directory, 7 files
```
helm chart 
```
# cat Chart.yaml
apiVersion: v2
name: dev
description: python chart
type: application
version: 0.1.0
appVersion: 1.16.0
helm values.yaml 定义项目变量值

# cat values.yaml
dp:
  namespace: dev
  replicaCount: 1
  imagePullSecrets: aliyun-docker-registry
  gpu_enable: false
  containers:
    image:


    readinessProbe:
      tcpSocket:
        initialDelaySeconds: 60


    livenessProbe:
      tcpSocket:
        initialDelaySeconds: 120


    env:
      USE_APOLLO: "1"
      APOLLO_APP_ID: ocr-server
      APOLLO_SERVER_URL: http://qasapollo.domain.com:8080
      APOLLO_CLUSTER: default
      APP_WORKDIR : /root
      APP_LOG_PATH : /root/logs
      OUTPUT_REDIRECT : /dev/stdout
      TOOLS_PATH : /root/tools


    cont1:
      name: ai-coeus-admin
      port: 5000
      command: ["sh"]
      args: ["start_admin1.sh"]
      resources:
        limits:
          cpu: 250m
          memory: 512Mi
        requests:
          cpu: 250m
          memory: 512Mi
      resourcesGPU:
        limits:
          nvidia.com/gpu: 1
      volumeMounts:
        /root/logs: vol1
        /root/tools: vol2
        /root/data/models: vol3
      


  volumes:
    hostPath:
      path: /share_log_mount/
      tools_path: /share_log_mount/tools
      models_path: /share_log_mount/models


      
service:
  type: ClusterIP
  ser1:
    port: 8080
      


ingress:
  enabled: true
  annotations:
    rewrite_target: /$2
  host: coeus.domain.com.cn
  http:
    paths:
      path: /admin


#  tls:
#   secretName: insnail-images
```

helm _helpers.tpl  定义 helm 嵌套模板，可被其他模板引用
```
# cat _helpers.tpl
{{/*
资源名字
*/}}
{{- define "name" -}}
{{ .Release.Name }}
{{- end -}}


{{/*
资源标签
*/}}
{{- define "labels" -}}
app: {{ template "name" .}}
release: {{ .Release.Name }}
{{- end -}}


{{/*
Pod标签
*/}}
{{- define "selectlabels" -}}
app: {{ template "name" .}}
{{- end -}}
```
helm deployment.yaml 定义 k8s pod 部署模板
```
# cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "name" . }}
  labels:
    {{- include "labels" . | nindent 4 }}
  namespace: {{ .Values.dp.namespace }}
spec:
  replicas: {{ .Values.dp.replicaCount }}
  selector:
    matchLabels:
      {{- include "selectlabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "selectlabels" . | nindent 8 }}
    spec:
      {{- if .Values.dp.gpu_enable }}
      nodeSelector:
        aliyun.accelerator/nvidia_name: Tesla-P4
      {{- else }}
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: aliyun.accelerator/nvidia_name
                operator: DoesNotExist
      {{- end }}
      imagePullSecrets:
      - name: {{ .Values.dp.imagePullSecrets }}
      containers:
        - name: {{ .Values.dp.containers.cont1.name }}
          image: {{ .Values.dp.containers.image }}
          imagePullPolicy: IfNotPresent
          ports:
            - name: port
              containerPort: {{ .Values.dp.containers.cont1.port }}
              protocol: TCP
          command: {{ .Values.dp.containers.cont1.command }}
          args: {{ .Values.dp.containers.cont1.args }}
          readinessProbe:
            failureThreshold: 3
            tcpSocket:
              port: {{ .Values.dp.containers.cont1.port }}
            initialDelaySeconds: {{ .Values.dp.containers.readinessProbe.tcpSocket.initialDelaySeconds }}
            periodSeconds: 2
            successThreshold: 2
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            tcpSocket:
              port: {{ .Values.dp.containers.cont1.port }}
            initialDelaySeconds: {{ .Values.dp.containers.livenessProbe.tcpSocket.initialDelaySeconds }}
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 3
          resources:
            {{- if .Values.dp.gpu_enable }}
            {{- toYaml .Values.dp.containers.cont1.resourcesGPU | nindent 12 }}
            {{- else }}
            {{- toYaml .Values.dp.containers.cont1.resources | nindent 12 }}
            {{- end }}
          env:
          {{- with .Values.dp.containers.env }}
           {{- range $key,$val := . }}
            - name : {{ $key }}
              value : {{ $val | quote }}
           {{- end}}
          {{- end }}
          volumeMounts:
          {{- with .Values.dp.containers.cont1.volumeMounts }}
          {{- range $key,$val := . }}
          - mountPath: {{ $key }}
            name: {{ $val }}
          {{- end}}
          {{- end }}
      volumes:
      - name: vol1
        hostPath:
          path: {{ .Values.dp.volumes.hostPath.path }}
      - name: vol2
        hostPath:
          path: {{ .Values.dp.volumes.hostPath.tools_path }}
      - name: vol3
        hostPath:
          path: {{ .Values.dp.volumes.hostPath.models_path }}
```
helm service.yaml 定义 k8s 负载均衡
```
# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "name" . }}
  labels:
    {{- include "labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - name: {{ .Values.dp.containers.cont1.name }}
      port: {{ .Values.service.ser1.port }}
      targetPort: {{ .Values.dp.containers.cont1.port }}
      protocol: TCP
  selector:
    {{- include "selectlabels" . | nindent 4 }}
```
helm ingress.yaml 定义 k8s 服务发现访问
```
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    {{- if ne .Values.ingress.http.paths.path "/" }}
    nginx.ingress.kubernetes.io/rewrite-target: {{ .Values.ingress.annotations.rewrite_target }}
    {{- end }}
#    kubernetes.io/ingress.class: nginx
#    nginx.ingress.kubernetes.io/rewrite-target: /$2
#    nginx.ingress.kubernetes.io/configuration-snippet: |
#      proxy_set_header Upgrade "websocket";
#      proxy_set_header Connection "Upgrade";
  name: {{ include "name" . }}
  labels:
    {{- include "labels" . | nindent 4 }}
spec:
#{{- if .Values.ingress.tls }}
#  tls:
#    - hosts:
#    {{- .Values.ingress.host | nindent 4  }}
#      secretName: {{ .Values.ingress.tls.secretName }}
#{{- end }}
  rules:
  - host: {{ .Values.ingress.host | quote }}
    http:
      paths:
      - backend:
          serviceName: {{ include "name" . }}
          servicePort: {{ .Values.service.ser1.port }}
        path: {{ .Values.ingress.http.paths.path }}
{{- end }}
```

