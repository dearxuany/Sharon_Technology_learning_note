# es 集群优化_elasticsearch 节点重启后报 master not discovered or elected yet 无法选举 master 问题解决
## 问题背景
elasticsearch 单个节点维护后，发现维护节点长时间无法加入到已有集群，无法获取 master 信息。</br>
报错信息：master not discovered or elected yet, an election requires at least 2 nodes
```
[2020-04-28T15:35:47,364][WARN ][o.e.c.c.ClusterFormationFailureHelper] [gzyw53-opd-skywalking-03] master not discovered or elected yet, an election requires at least 2 nodes with ids from [8FSE6GA1QLqsqvn6iEbWIw, R2AXVk0nRZiVeVZhxdv5iA, Db4FW6v0T26o-IBacRSv2Q], have discovered [] which is not a quorum; discovery will continue using [10.0.0.57:19300, 10.0.0.58:19300] from hosts providers and [{gzyw53-opd-skywalking-03}{8FSE6GA1QLqsqvn6iEbWIw}{om-b44LlSteEbQRe60uEPw}{10.0.0.59}{10.0.0.59:19300}{ml.machine_memory=8201506816, xpack.installed=true, ml.max_open_jobs=20}, {gzyw53-opd-skywalking-01.snail}{Db4FW6v0T26o-IBacRSv2Q}{oWTFN9HlRECFtJwuDd1Cqw}{10.0.0.57}{10.0.0.57:19300}{ml.machine_memory=16657219584, ml.max_open_jobs=20, xpack.installed=true}, {gzyw53-opd-skywalking-02.snail}{R2AXVk0nRZiVeVZhxdv5iA}{HUCWDEyxRZyu9Hyg422b5w}{10.0.0.58}{10.0.0.58:19300}{ml.machine_memory=8201506816, ml.max_open_jobs=20, xpack.installed=true}] from last-known cluster state; node term 30, last-accepted version 1842 in term 30
```
已有集群信息：集群现有节点为 2，由于此前发生故障故 status 为 red，有未分配的分片
```
$ curl http://10.0.0.57:19200/_cluster/health?pretty=true
{
  "cluster_name" : "gzyw53-opd-es-c2",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 823,
  "active_shards" : 855,
  "relocating_shards" : 0,
  "initializing_shards" : 6,
  "unassigned_shards" : 1348,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 103,
  "number_of_in_flight_fetch" : 189,
  "task_max_waiting_in_queue_millis" : 1066730,
  "active_shards_percent_as_number" : 38.70529651425984
}
```

## 问题分析
集群一共三个节点 10.0.0.57、10.0.0.58、10.0.0.59，各节点的机器初始化配置均为以下所示：
```
#discovery.seed_hosts: ["host1", "host2"]
discovery.seed_hosts: ["10.0.0.57:19300", "10.0.0.58:19300", "10.0.0.59:19300"]

# Bootstrap the cluster using an initial set of master-eligible nodes:
cluster.initial_master_nodes: ["gzyw53-opd-skywalking-01"]

discovery.zen.minimum_master_nodes: 2
discovery.zen.ping_timeout: 120s
```
* discovery.seed_hosts 为集群种子节点，描述了集群内的所有节点信息；
* cluster.initial_master_nodes 为集群初始化选定的 master 节点，设置该信息可让集群在形成时快速选举 master 节点；
* discovery.zen.minimum_master_nodes 为形成单个集群的节点最少有选举为 master 资格的节点数量，一般为 (集群总节点数 n+1)/2，防止脑裂。</br>

此处，3个节点初始化集群 master 均为 gzyw53-opd-skywalking-01，可选举 master 节点数量为 (3+1)/2=2。</br>
集群在初始化时，master 节点确实选举为 gzyw53-opd-skywalking-01，但维护过程中 gzyw53-opd-skywalking-01 曾经下线，集群自动将 master 节点变更为了其他主机。主要有两个条件：

* 当维护节点再次加入到集群时，当前已有集群的 master 和重启时设置的 cluster.initial_master_nodes 不匹配，故无法将自身选举为主节点； 
* 且当前未加入已有集群的流动节点数量为 1，不满足 discovery.zen.minimum_master_nodes 为 2 的条件，故当前节点无法独立形成单个集群；</br>


故该离线节点一直处于寻找集群以及选举 master 的过程中，无法加入已有集群。


## 解决办法
修改离线节点的 cluster.initial_master_nodes 为已有集群的 master 节点，缩短 master 选举过程。</br>
</br>
查找当前集群的 master 节点
```
$ curl http://10.0.0.57:19200/_cat/nodes?v
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.0.0.59           61          98   5    6.44    5.43     4.78 mdi       *      gzyw53-opd-skywalking-03
10.0.0.57            3          82   3    0.84    0.83     1.15 mdi       -      gzyw53-opd-skywalking-01
```
将离线节点的 cluster.initial_master_nodes 改为已有集群的 master 节点
```
# Bootstrap the cluster using an initial set of master-eligible nodes:
cluster.initial_master_nodes: ["gzyw53-opd-skywalking-03"]
```
查看集群状态，可见集群节点数量从 2 快速地变为了 3，开始进行分片分配
```
$ curl http://10.0.0.57:19200/_cluster/health?pretty=true
{
  "cluster_name" : "gzyw53-opd-es-c2",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 859,
  "active_shards" : 904,
  "relocating_shards" : 0,
  "initializing_shards" : 6,
  "unassigned_shards" : 1299,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 111,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 1220048,
  "active_shards_percent_as_number" : 40.92349479402444
}


[skywalking@gzyw53-opd-skywalking-01 skywalking]$ curl http://10.0.0.57:19200/_cluster/health?pretty=true
{
  "cluster_name" : "gzyw53-opd-es-c2",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 859,
  "active_shards" : 904,
  "relocating_shards" : 0,
  "initializing_shards" : 8,
  "unassigned_shards" : 1297,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 112,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 1234033,
  "active_shards_percent_as_number" : 40.92349479402444
}
```
