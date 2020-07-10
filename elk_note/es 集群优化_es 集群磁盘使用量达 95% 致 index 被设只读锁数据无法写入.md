# es 集群优化_es 集群磁盘使用量达 95% 致 index 被设只读锁数据无法写入
## 背景
logstash 写入 es 出现大量报错，数据无法写入到 es
```
[2019-07-01T14:01:15,044][INFO ][logstash.outputs.elasticsearch] retrying failed action with response code: 403 ({"type"=>"cluster_block_exception", "reason"=>"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"})
```
## 解决办法
先清理磁盘到95%一下，再kibana 上操作解锁，es不会自动解锁
```
PUT /_all/_settings
{"index.blocks.read_only_allow_delete": null}
```
返回
```
{
  "acknowledged" : true
}
```
