# es 集群滚动更新
https://www.elastic.co/guide/cn/elasticsearch/guide/current/_rolling_restarts.html

## 滚动更新操作
* 禁止分片分配
```
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"
    }}

# 返回结果

{
  "acknowledged" : true,
  "persistent" : { },
  "transient" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "enable" : "none"
        }
      }
    }
  }
}
```
* 关闭单个节点
* 执行维护升级
* 重启节点，确认维护节点已加入集群
* 重启分片分配
```
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }}
```
* 查看集群设置
```
GET /_cluster/settings

{
  "persistent" : {
    "xpack" : {
      "monitoring" : {
        "collection" : {
          "enabled" : "true"
        }
      }
    }
  },
  "transient" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "enable" : "all"
        }
      }
    }
  }
}
```
* 查看分片集群健康状态
```
GET _cluster/health?pretty=true

{
  "cluster_name" : "elasticsearch-C1",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1466,
  "active_shards" : 2932,
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
分片再平衡会花一些时间，一直等到集群变成 绿色 状态后再继续。
