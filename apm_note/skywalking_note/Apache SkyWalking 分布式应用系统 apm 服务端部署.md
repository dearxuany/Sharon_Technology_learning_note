# Apache SkyWalking 分布式应用系统 apm 服务端部署
skywalking 结构
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/apm_images/skywalking_Image_01.png)</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/apm_images/skywalking_Image_02.png)
## 依赖
此处依赖均使用 ansible playbook 批量安装
### java
```
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```
### elasticsearch 7.1.1
由于 skywalking 写入 es 的索引较多（主要支持历史数据保留），而 es 7 之后单个 es 节点限制 1000 个分片，故需要部署三节点 es 集群保证 skywalking 数据写入。
```
# curl http://10.0.0.57:19200/_cluster/health?pretty=true
{
  "cluster_name" : "gz-elasticsearch-02",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 2,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```
除基础集群配置外，为保证 skywalking 写入到 es 的速度，需设置 threadpool write queue_size 优化参数
```
# vim /etc/elasticsearch/elasticsearch.yml

thread_pool.write.queue_size: 10000
```
### kibana
es 集群可视化界面，此处主要用于监控 es 索引分片状态等。


## skywalking 

### 安装包
后端存储数据库使用 elasticsearch 7.1，对应下载 skywalking es 源码包 </br>
http://skywalking.apache.org/downloads/  </br>
https://github.com/apache/skywalking/blob/master/docs/en/setup/README.md
```
# wget https://mirror.bit.edu.cn/apache/skywalking/7.0.0/apache-skywalking-apm-es7-7.0.0.tar.gz
# tar -xvzf ./apache-skywalking-apm-es7-7.0.0.tar.gz
# mv ./apache-skywalking-apm-bin-es7 /sdata/usr/local/skywalking
```
目录结构
```
# tree -L 1 .
.
├── agent
├── bin     # 可执行文件，启动脚本等
├── config  # skywalking 应用配置、告警配置、日志配置
├── LICENSE
├── licenses
├── NOTICE
├── oap-libs
├── README.txt
├── tools
└── webapp  # UI 及前端配置

7 directories, 3 files
```
### skywalking 配置 
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/backend/backend-ui-setup.md
```
# vim /sdata/usr/local/skywalking/config/application.yml
```
#### 集群模式配置
此处选取单节点模式
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/backend/backend-cluster.md
```
cluster:
  selector: ${SW_CLUSTER:standalone}
  standalone:
```
#### 绑定端口、ip、数据处理周期配置
skywalking默认12800为rest api通信端口，默认11800为gRPC api通信端口，控制台默认监听 8080 端口</br>
注意：关闭 SSL 加密，数据处理周期部分会被 es 存储设置部分的周期覆盖。</br>
```
core:
  selector: ${SW_CORE:default}
  default:
    # Mixed: Receive agent data, Level 1 aggregate, Level 2 aggregate
    # Receiver: Receive agent data, Level 1 aggregate
    # Aggregator: Level 2 aggregate
    role: ${SW_CORE_ROLE:Mixed} # Mixed/Receiver/Aggregator
    restHost: ${SW_CORE_REST_HOST:10.0.0.57}
    restPort: ${SW_CORE_REST_PORT:12800}
    restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
    gRPCHost: ${SW_CORE_GRPC_HOST:10.0.0.57}
    gRPCPort: ${SW_CORE_GRPC_PORT:11800}
       #gRPCSslEnabled: ${SW_CORE_GRPC_SSL_ENABLED:false}
    #gRPCSslKeyPath: ${SW_CORE_GRPC_SSL_KEY_PATH:""}
    #gRPCSslCertChainPath: ${SW_CORE_GRPC_SSL_CERT_CHAIN_PATH:""}
    #gRPCSslTrustedCAPath: ${SW_CORE_GRPC_SSL_TRUSTED_CA_PATH:""}


    downsampling:
      - Hour
      - Day
      - Month

    enableDataKeeperExecutor: ${SW_CORE_ENABLE_DATA_KEEPER_EXECUTOR:true} # Turn it off then automatically metrics data delete will be close.
    dataKeeperExecutePeriod: ${SW_CORE_DATA_KEEPER_EXECUTE_PERIOD:5} # How often the data keeper executor runs periodically, unit is minute
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:90} # Unit is minute
    minuteMetricsDataTTL: ${SW_CORE_MINUTE_METRIC_DATA_TTL:90} # Unit is minute
    hourMetricsDataTTL: ${SW_CORE_HOUR_METRIC_DATA_TTL:36} # Unit is hour
    dayMetricsDataTTL: ${SW_CORE_DAY_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_CORE_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Cache metric data for 1 minute to reduce database queries, and if the OAP cluster changes within that minute,
    # the metrics may not be accurate within that minute.
    enableDatabaseSession: ${SW_CORE_ENABLE_DATABASE_SESSION:true}
    topNReportPeriod: ${SW_CORE_TOPN_REPORT_PERIOD:10} # top_n record worker report cycle, unit is minute
    # Extra model column are the column defined by in the codes, These columns of model are not required logically in aggregation or further query,
    # and it will cause more load for memory, network of OAP and storage.
    # But, being activated, user could see the name in the storage entities, which make users easier to use 3rd party tool, such as Kibana->ES, to query the data by themselves.
    activeExtraModelColumns: ${SW_CORE_ACTIVE_EXTRA_MODEL_COLUMNS:false}
```



#### 后端数据存储设置
默认使用H2作为数据库，但H2经重启后丢失数据，故配置使用 elasticsearch 7 </br>
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/backend/backend-storage.md</br>
注释除 es7 外所有存储配置，启用elasticsearch7配置
```
storage:
  selector: ${SW_STORAGE:elasticsearch7}
  elasticsearch7:
    #nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:10.0.0.57:19200,10.0.0.58:19200,10.0.0.59:19200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    #trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    #trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    enablePackedDownsampling: ${SW_STORAGE_ENABLE_PACKED_DOWNSAMPLING:true} # Hour and Day metrics will be merged into minute index.
    dayStep: ${SW_STORAGE_DAY_STEP:1} # Represent the number of days in the one minute/hour/day index.
    #user: ${SW_ES_USER:""}
    #password: ${SW_ES_PASSWORD:""}
    #secretsManagementFile: ${SW_ES_SECRETS_MANAGEMENT_FILE:""} # Secrets management file in the properties format includes the username, password, which are managed by 3rd party tool.
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Those data TTL settings will override the same settings in core module.
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:4000} # Execute the bulk every 1000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:40}
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:30} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:4} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:8000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    profileTaskQueryMaxSize: ${SW_STORAGE_ES_QUERY_PROFILE_TASK_SIZE:200}
```

需要注意  dayStep 以及数据保留 TTL 的配置</br>
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/backend/ttl.md
```
Daily index step(storage/elasticsearch/dayStep, default 1) represents the index creation period. In this period, several days(dayStep value)' metrics are saved.
Mostly, users don't need to change the value manually. As SkyWalking is designed to observe large scale distributed system. But in some specific cases, users want to set a long TTL value, such as more than 60 days, but their ElasticSearch cluster isn't powerful due to the low traffic in the production environment. This value could be increased to 5(or more), if users could make sure single one index could support these days(5 in this case) metrics and traces.
Such as, if dayStep == 11,
1. data in [2000-01-01, 2000-01-11] will be merged into the index-20000101.
2. data in [2000-01-12, 2000-01-22] will be merged into the index-20000112.

NOTICE, TTL deletion would be affected by these. You should set an extra more dayStep in your TTL. Such as you want to TTL == 30 days and dayStep == 10, you actually need to set TTL = 40;
```
#### receiver 配置
https://github.com/apache/skywalking/blob/v7.0.0/docs/en/setup/backend/backend-receivers.md
```
receiver-sharing-server:
  selector: ${SW_RECEIVER_SHARING_SERVER:default}
  default:
    authentication: ${SW_AUTHENTICATION:""}


receiver-register:
  selector: ${SW_RECEIVER_REGISTER:default}
  default:


receiver-trace:
  selector: ${SW_RECEIVER_TRACE:default}
  default:
    bufferPath: ${SW_RECEIVER_BUFFER_PATH:/sdata/data/skywalking/trace-buffer/}  # Path to trace buffer files, suggest to use absolute path
    bufferOffsetMaxFileSize: ${SW_RECEIVER_BUFFER_OFFSET_MAX_FILE_SIZE:100} # Unit is MB
    bufferDataMaxFileSize: ${SW_RECEIVER_BUFFER_DATA_MAX_FILE_SIZE:500} # Unit is MB
    bufferFileCleanWhenRestart: ${SW_RECEIVER_BUFFER_FILE_CLEAN_WHEN_RESTART:false}
    sampleRate: ${SW_TRACE_SAMPLE_RATE:10000} # The sample rate precision is 1/10000. 10000 means 100% sample in default.
    slowDBAccessThreshold: ${SW_SLOW_DB_THRESHOLD:default:200,mongodb:100} # The slow database access thresholds. Unit ms.


receiver-jvm:
  selector: ${SW_RECEIVER_JVM:default}
  default:


receiver-clr:
  selector: ${SW_RECEIVER_CLR:default}
  default:


receiver-profile:
  selector: ${SW_RECEIVER_PROFILE:default}
  default:


service-mesh:
  selector: ${SW_SERVICE_MESH:default}
  default:
    bufferPath: ${SW_SERVICE_MESH_BUFFER_PATH:/sdata/data/skywalking/mesh-buffer/}  # Path to trace buffer files, suggest to use absolute path
    bufferOffsetMaxFileSize: ${SW_SERVICE_MESH_OFFSET_MAX_FILE_SIZE:100} # Unit is MB
    bufferDataMaxFileSize: ${SW_SERVICE_MESH_BUFFER_DATA_MAX_FILE_SIZE:500} # Unit is MB
    bufferFileCleanWhenRestart: ${SW_SERVICE_MESH_BUFFER_FILE_CLEAN_WHEN_RESTART:false}


istio-telemetry:
  selector: ${SW_ISTIO_TELEMETRY:default}
  default:


envoy-metric:
  selector: ${SW_ENVOY_METRIC:default}
  default:
    alsHTTPAnalysis: ${SW_ENVOY_METRIC_ALS_HTTP_ANALYSIS:""}
```

注意新增 trace-buffer 和 mesh-buffer 的路径
```
mkdir -p /sdata/data/skywalking/trace-buffer/
mkdir -p /sdata/data/skywalking/mesh-buffer/
```
#### 其他配置
```
query:
  selector: ${SW_QUERY:graphql}
  graphql:
    path: ${SW_QUERY_GRAPHQL_PATH:/graphql}


alarm:
  selector: ${SW_ALARM:default}
  default:


telemetry:
  selector: ${SW_TELEMETRY:none}
  none:

configuration:
  selector: ${SW_CONFIGURATION:none}
  none:
```

#### UI 前端配置
skywalking 7.0.0 webapp 由于考虑安全性问题已剥离用户系统功能，需使用 nginx 额外配置权限管理
```
# vim /sdata/usr/local/skywalking/webapp/webapp.yml

server:
  port: 8080


collector:
  path: /graphql
  ribbon:
    ReadTimeout: 10000
    # Point to all backend's restHost:restPort, split by ,
    listOfServers: 10.0.0.57:12800
```
#### jvm 启动配置
后端内存、日志路径调整
```
# vim bin/oapService.sh

OAP_LOG_DIR="${OAP_LOG_DIR:-/sdata/var/log/skywalking}"
JAVA_OPTS=" -Xms2g -Xmx2g"
```
前端内存调整
```
# vim bin/webappService.sh

WEBAPP_LOG_DIR="${WEBAPP_LOG_DIR:-/sdata/var/log/skywalking}"
JAVA_OPTS=" -Xms256M -Xmx512M"
```
## skywalking 启动测试
新增 skywalking 用户并修改相关路径权限
```
# chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
# useradd skywalking
# chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow

# chown -R skywalking:skywalking /sdata/var/log/skywalking
# chown -R skywalking:skywalking /sdata/data/skywalking
# chown -R skywalking:skywalking /sdata/usr/local/skywalking
```
启动 skywalking
```
# sh bin/startup.sh
SkyWalking OAP started successfully!
SkyWalking Web Application started successfully!
```
查看监听端口
```
# netstat -tunpl|grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      30532/java 

# netstat -tunpl|grep 11800
tcp6       0      0 10.0.0.57:11800         :::*                    LISTEN      2207/java  

# netstat -tunpl|grep 12800
tcp6       0      0 10.0.0.57:12800         :::*                    LISTEN      2207/java
```
访问 skywalking 页面
```
http://10.0.0.57:8080
```
注意： 如果skywalking是初次连接elasticsearch服务，启动会比较慢，因为skywalking需要向es服务创建很多的index。所以在未创建完成之前，访问这个页面会是空白的。此时可以通过查看日志来判断启动是否完成。
```
# tail -f skywalking-oap-server.log
```
