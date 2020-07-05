# es 集群优化_磁盘水印低 Low disk watermark 导致集群节点分片分配不均匀问题解决
## 背景
发现 es 集群节点分片不均匀，其中一个节点被分配的分片很少，且有出现了很多 unassigned shards，节点磁盘上还有上百 GB 空间。


## 原因
如果没有足够的磁盘空间节点，主节点可能无法分配分片（它不会将分片分配给使用率超过85％的磁盘的节点）。一旦节点达到此磁盘使用级别，或Elasticsearch称为“低磁盘水印”，将不会为其分配更多分片。
https://www.cnblogs.com/yfb918/p/10475083.html
```
# 查看节点磁盘负载

GET /_cat/allocation

162 206.1gb 1.5tb 212.5gb 1.7tb 88 10.0.0.153 10.0.0.153 gzyw53-elasticsearch-02
999   1.4tb 1.4tb 267.9gb 1.7tb 85 10.0.0.154 10.0.0.154 gzyw53-elasticsearch-03
260 430.4gb 1.5tb 191.6gb 1.7tb 89 10.0.0.152 10.0.0.152 gzyw53-elasticsearch-01
1399                                                      UNASSIGNED
```
此处，单节点磁盘大小为 1.7 TB， Low disk watermark 为 1.7 * 0.85 = 1.45 TB，即当节点磁盘可用量少于 261 GB 时，主节点会对该节点停止分配分片。故导致节点分片少，出现大量未分配分片。

## 解决办法
如果您的节点具有大磁盘容量，则85％的低水印可能太低。您可以使用群集更新设置API进行更改cluster.routing.allocation.disk.watermark.low或cluster.routing.allocation.disk.watermark.high。例如，此Stack Overflow线程指出，如果您的节点具有5TB磁盘容量，则可以安全地将低磁盘水印增加到90％。
```
# dev tool 设置
PUT /_cluster/settings
{
    "transient": {    
          "cluster.routing.allocation.disk.watermark.low": "90%"
    }
}

# 返回
{
  "acknowledged" : true,
  "persistent" : { },
  "transient" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "disk" : {
            "watermark" : {
              "low" : "90%"
            }
          }
        }
      }
    }
  }
}
```

如果希望在群集重新启动时保持配置更改，请将“transient”替换为“persistent”，或者在配置文件中更新这些值。您可以选择使用字节或百分比值来更新这些设置，但请务必记住Elasticsearch文档中的这一重要说明：“百分比值是指已用磁盘空间，而字节值是指可用磁盘空间。”
https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-cluster.html#disk-based-shard-allocation
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
          "disk" : {
            "watermark" : {
              "low" : "90%"
            }
          },
          "enable" : "all"
        }
      }
    }
  }
}
```
注意：会导致部分分片重新分片，均衡分片过程中，易导致集群节点负载过高故障
