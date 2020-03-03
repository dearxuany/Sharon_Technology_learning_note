# elk 集群_ansible playbook 部署(3)_kafka 消息队列
生产 filebeat 节点位于阿里云生产环境内，而 logstash 及 es 集群使用本地办公网主机资源，中间经过较长的公网传输。
为防止数据丢失、安全性、性能考虑，需在 filebeat 及 logstash 间加一层 kafka 消息缓存，且需要公网加密传输(减少vpn带宽消耗)。
## kafka 安装
* 需要安装 java 8 以上版本 java
* kafka 依赖zookeeper，已经包含在kafka的tar包里，不需要额外下载安装
* 阿里云 kafka 监听端口 19092，安全组开放此端口给办公网出口 ip 访问
* kafka 和内置的 zookeeper 各自需要有独立的日志和数据目录，不能与其他项目共用，否则无法启动 kafka</br>

注：由于此处使用单节点，所以此处直接使用命令部署
### 源码安装
解压源码包
```
# tar -xvz -f /sdata/usr/local/src/kafka_2.11-1.1.0.tgz -C /sdata/usr/local/
# mv ./kafka_2.11-1.1.0 ./kafka
# chown -R elk:elk /sdata/usr/local/kafka
```
### zookeeper 配置
创建 zookeeper data 及 log 目录
```
# mkdir -p /sdata/var/log/zookeeper
# chown elk:elk /sdata/var/log/zookeeper
# mkdir -p /sdata/data/zookeeper
# chown elk:elk /sdata/data/zookeeper
```
创建 myid 文件，必须放置在 zookeeper data 目录下，myid 的内容必须和配置文件中设置的服务器 id 一致
```
# echo 1 > /sdata/data/zookeeper/myid
# chown elk:elk /sdata/data/zookeeper/myid
```
配置 zookeeper 配置文件：
```
# vim /sdata/usr/local/kafka/config/zookeeper.properties
# 生产者链接使用默认端口
clientPort=2181
maxClientCnxns=0
# 日志及数据路径
dataDir=/sdata/data/zookeeper
dataLogDir=/sdata/var/log/zookeeper
#最大客户端连接数
maxClientCnxns=100
#Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，单位为 ms
tickTime=2000
#各节点和主节点链接初始化超时时间 60*2000 ms=120s
initLimit=60
#各节点和主节点同步时间间隔 60*2000 ms=120s
syncLimit=60
#server.myid=ip:followers_connect to the leader:leader_election，节点通信端口2888，首领选举端口3888
server.1=172.18.65.46:2888:3888
```
### kafka 配置
创建 kafka 数据及日志目录
```
# mkdir -p /sdata/var/log/kafka
# chown elk:elk /sdata/var/log/kafka
# mkdir -p /sdata/data/kafka
# chown elk:elk /sdata/data/kafka
```
修改 kafka 配置
注：此处为方便调试暂时不配加密，kafka 正常通信后再新加 ssl 加密。</br>
```
# vim /sdata/usr/local/kafka/config/server.properties

# 使用默认 broker id
broker.id=0
# 消费者（logstash）内网监听主机及端口
listeners=PLAINTEXT://alihn1-opd-elk-01.snail:9092
# 消费者（logstash）外网监听主机及端口
advertised.listeners=PLAINTEXT://alihn1-opd-elk-01.snail:19092
# 接收及发送网络信息线程数
num.network.threads=3
# 请求处理线程数
num.io.threads=8
#套接字服务器使用的发送缓冲区(SO_SNDBUF)
socket.send.buffer.bytes=102400
#套接字服务器使用的接收缓冲区(SO_RCVBUF)
socket.receive.buffer.bytes=102400
#套接字服务器将接受的请求的最大大小(防止OOM)。
socket.request.max.bytes=104857600
# 日志路径
log.dirs=/sdata/var/log/kafka
# 新创建 topic 包含分区数量，只能增多不能减少，集群模式下分区数应大于 broker 数（待调优）
num.partitions=1
# 单个数据目录分配线程数 (待调优)
num.recovery.threads.per.data.dir=1
# kafka 内部 topics 配置（待调优）
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
# 数据在 kafka 内保留时间，默认168小时即一周，填满一个日志片段才开始计算过期时间
log.retention.hours=168
# 日志片段大小，当日志片段满 1GB 后开启新日志片段，旧日志片段等待过期
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
# Zookeeper 配置
zookeeper.connect=172.18.65.46:2181
zookeeper.connection.timeout.ms=6000
```
内外网的监听不能写 ip，因为如果写 ip 内网 listeners 必须写内网 ip，而外网 advertised.listeners 必须写 ecs 的公网 ip，否则外网 logstash 无法消费。这又导致生产和消费出现如下所示报错，大概是因为阿里云内网的主机无法用内网 ip 找到 kafka 的 leader，但又无法使用外网 ip 去链接在同一个 vpc 中的 kafka，所以必须将 kafka 设置为按照主机名和端口链接。</br>

当在阿里云 vpc 中链接时，hostname 解析为 vpc 内网 ip。当链接来源于vpc外时，将 hostname 解析为 ecs 或 NAT 网关公网 ip 。对于阿里云上的主机使用阿里云的 DNS 主机名解析，对于本地内网的消费者 logstash 则直接在该主机上编写 hostname 指向 kafka 主机或 NAT 网关的公网 ip。</br>

### 阿里云平台配置 kafka 节点 ECS 安全策略
阿里云控制台开放 ecs 所在安全组入方向的19092，允许本地网络的出口 ip 及 阿里云 vpc ip 访问。</br>
注意：还需要在本地网络的网关上配置端口映射</br>

### 启动 kafka
先启动 zookeeper 后启动 kafka
#### zookeeper 启动验证
启动 zookeeper
```
nohup /sdata/usr/local/kafka/bin/zookeeper-server-start.sh  /sdata/usr/local/kafka/config/zookeeper.properties  >>/sdata/var/log/zookeeper/zookeeper-start.log 2>&1 &
```
查看 zookeeper 进程及端口
```
ps -ef|grep zookeeper
netstat -tunpl|grep 2181
(Not all processes could be identified, non-owned process info
will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:2181            0.0.0.0:*               LISTEN      21691/java  
```
使用 kafka 内置工具验证
```
$ sh bin/zookeeper-shell.sh 172.18.65.46:2181
Connecting to 172.18.65.46:2181
Welcome to ZooKeeper!
JLine support is disabled


WATCHER::


WatchedEvent state:SyncConnected type:None path:null
```
#### kafka 启动验证
启动 kafka
```
nohup /sdata/usr/local/kafka/bin/kafka-server-start.sh /sdata/usr/local/kafka/config/server.properties >>/sdata/var/log/kafka/kafka-start.log 2>&1 &
```
验证进程及端口，kafka 完成启动一共产生两个进程，一个 zookeeper，一个 kafka
```
$ ps -ef|grep kafka
$ netstat -tunpl|grep 19092
(Not all processes could be identified, non-owned process info
will not be shown, you would have to be root to see it all.)
tcp        0      0 172.18.65.46:19092      0.0.0.0:*               LISTEN      1112/java
```
添加测试 topic
```
$ /sdata/usr/local/kafka/bin/kafka-topics.sh --create --zookeeper alihn1-opd-elk-01.snail:2181 --replication-factor 1 --partitions 1 --topic kafkatest
Created topic "kafkatest".
```
查看 topic 名单
```
$ /sdata/usr/local/kafka/bin/kafka-topics.sh --zookeeper alihn1-opd-elk-01.snail:2181 --list
kafkatest
```
