# es 数据迁移_elasticsearch-dump 全量迁移 es 数据
## es 数据迁移方式
如果业务可以停服或者可以暂停写操作，可以使用以下几种方式进行数据迁移：
* elasticsearch-dump
* snapshot
* reindex
* logstash

## elasticsearch-dump 全量迁移 es 数据
官网 https://github.com/taskrabbit/elasticsearch-dump
使用简介 https://www.jianshu.com/p/50ef4c9090f0
```
# docker pull taskrabbit/elasticsearch-dump

# docker images
REPOSITORY                                                     TAG                 IMAGE ID            CREATED             SIZE
taskrabbit/elasticsearch-dump                                  latest              0e4ee0f8dfde        24 hours ago        150MB
```
正确迁移顺序：</br>
settings -> analyzer(已包含在 settings中，不需特别迁移) -> mapping -> data（可单独迁移，但不包含原集群 index 的 settings 和 mapping）</br>
</br>
注意第一条命令先将索引的settings先迁移，如果直接迁移mapping或者data将失去原有集群中索引的配置信息如分片数量和副本数量等，当然也可以直接在目标集群中将索引创建完毕后再同步mapping与data。
```
# 迁移 settings
# docker run --rm -ti taskrabbit/elasticsearch-dump   --input=http://elastic:passwd@10.0.0.149:9200/knowledge   --output=http://elastic:passwd@10.0.0.173:19201/knowledge   --type=settings

Mon, 27 Apr 2020 07:50:08 GMT | starting dump
Mon, 27 Apr 2020 07:50:08 GMT | got 1 objects from source elasticsearch (offset: 0)
Mon, 27 Apr 2020 07:50:23 GMT | sent 1 objects to destination elasticsearch, wrote 0
Mon, 27 Apr 2020 07:50:23 GMT | got 0 objects from source elasticsearch (offset: 1)
Mon, 27 Apr 2020 07:50:23 GMT | Total Writes: 0
Mon, 27 Apr 2020 07:50:23 GMT | dump complete

# 迁移 mapping
# docker run --rm -ti taskrabbit/elasticsearch-dump   --input=http://elastic:passwd@10.0.0.149:9200/knowledge   --output=http://elastic:passwd@10.0.0.173:19201/knowledge   --type=mapping

Mon, 27 Apr 2020 07:51:38 GMT | starting dump
Mon, 27 Apr 2020 07:51:38 GMT | got 1 objects from source elasticsearch (offset: 0)
Mon, 27 Apr 2020 07:51:39 GMT | sent 1 objects to destination elasticsearch, wrote 1
Mon, 27 Apr 2020 07:51:39 GMT | got 0 objects from source elasticsearch (offset: 1)
Mon, 27 Apr 2020 07:51:39 GMT | Total Writes: 1
Mon, 27 Apr 2020 07:51:39 GMT | dump complete

# 迁移 data
# docker run --rm -ti taskrabbit/elasticsearch-dump   --input=http://elastic:passwd@10.0.0.149:9200/knowledge   --output=http://elastic:passwd@10.0.0.173:19201/knowledge   --type=data

Mon, 27 Apr 2020 07:53:51 GMT | starting dump
Mon, 27 Apr 2020 07:53:51 GMT | got 100 objects from source elasticsearch (offset: 0)
Mon, 27 Apr 2020 07:54:00 GMT | sent 100 objects to destination elasticsearch, wrote 100
Mon, 27 Apr 2020 07:54:00 GMT | got 100 objects from source elasticsearch (offset: 100)
Mon, 27 Apr 2020 07:54:06 GMT | sent 100 objects to destination elasticsearch, wrote 100

...


Mon, 27 Apr 2020 08:16:00 GMT | got 100 objects from source elasticsearch (offset: 12400)
Mon, 27 Apr 2020 08:16:06 GMT | sent 100 objects to destination elasticsearch, wrote 100
Mon, 27 Apr 2020 08:16:06 GMT | got 100 objects from source elasticsearch (offset: 12500)
Mon, 27 Apr 2020 08:16:12 GMT | sent 100 objects to destination elasticsearch, wrote 100
Mon, 27 Apr 2020 08:16:12 GMT | got 37 objects from source elasticsearch (offset: 12600)
Mon, 27 Apr 2020 08:16:50 GMT | sent 37 objects to destination elasticsearch, wrote 37
Mon, 27 Apr 2020 08:16:50 GMT | got 0 objects from source elasticsearch (offset: 12637)
Mon, 27 Apr 2020 08:16:50 GMT | Total Writes: 12637
Mon, 27 Apr 2020 08:16:50 GMT | dump complete
```
注意：</br>
* 在迁移 settings 和 mapping 基础上再迁移的 data 数据，数据量会比单独迁移 data 大很多
* elasticdump 会在迁移中记录 offset 防止重复迁移导致数据重复

## 别名设置
数据迁移完成后，需注意 index 的别名迁移
```
POST /_aliases
{
    "actions" : [
        { "add" : { "indices" : ["occupation_2020-04-23", "occupation_2020-04-24"], "alias" : "occupation" } }
    ]
}
```

