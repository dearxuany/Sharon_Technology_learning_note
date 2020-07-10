# es 集群优化_es 集群磁盘使用量达 95% 或单节点分片数达 1000 致 index 被设只读锁数据无法写入
## 背景
###  集群磁盘使用量达 95% 
logstash 写入 es 出现大量报错，数据无法写入到 es
```
[2019-07-01T14:01:15,044][INFO ][logstash.outputs.elasticsearch] retrying failed action with response code: 403 ({"type"=>"cluster_block_exception", "reason"=>"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"})
```
### 单节点分片数达 1000
根据官方解释，从Elasticsearch v7.0.0 开始，集群中的每个节点默认限制 1000 个shard，如果 es 集群有3个数据节点，那么最多 3000 shards。
```
[2020-02-20T21:14:59,610][WARN ][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"prd-bnail-2020.02.21", :_type=>"_doc", :routing=>nil}, #<LogStash::Event:0x632f1ec1>], :response=>{"index"=>{"_index"=>"prd-bnail-2020.02.21", "_type"=>"_doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"illegal_argument_exception", "reason"=>"Validation Failed: 1: this action would add [2] total shards, but this cluster currently has [3000]/[3000] maximum shards open;"}}}}
[2020-02-20T21:14:59,610][WARN ][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"prd-bnail-2020.02.21", :_type=>"_doc", :routing=>nil}, #<LogStash::Event:0x53ccb09d>], :response=>{"index"=>{"_index"=>"prd-bnail-2020.02.21", "_type"=>"_doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"illegal_argument_exception", "reason"=>"Validation Failed: 1: this action would add [2] total shards, but this cluster currently has [3000]/[3000] maximum shards open;"}}}}
[2020-02-20T21:14:59,610][WARN ][logstash.outputs.elasticsearch] Could not index event to Elasticsearch. {:status=>400, :action=>["index", {:_id=>nil, :_index=>"prd-bnail-2020.02.21", :_type=>"_doc", :routing=>nil}, #<LogStash::Event:0x6ad3156>], :response=>{"index"=>{"_index"=>"prd-bnail-2020.02.21", "_type"=>"_doc", "_id"=>nil, "status"=>400, "error"=>{"type"=>"illegal_argument_exception", "reason"=>"Validation Failed: 1: this action would add [2] total shards, but this cluster currently has [3000]/[3000] maximum shards open;"}}}}
```

## 解决办法
先清理磁盘到95%以下，再kibana 上操作解锁，es不会自动解锁。对于分片数超过 1000/node 的情况，需考虑迁移、增加节点、删除无用 index 再解锁。
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
