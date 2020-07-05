# es 集群优化_程序无法在 es 集群自动创建 index 问题解决
## 背景
项目程序上线后日志报错显示 index 不存在，且账号没有权限在 es 集群内新建 index，而已给程序开通 xpack 指定索引读写管理权限。
```
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index [channel-hub-2020.06] and [action.auto_create_index] contains [-*] which forbids automatic creation of the index","index_uuid":"_na_","index":"channel-hub-2020.06"}],"type":"index_not_found_exception","reason":"no such index [channel-hub-2020.06] and [action.auto_create_index] contains [-*] which forbids automatic creation of the index","index_uuid":"_na_","index":"channel-hub-2020.06"},"status":404}
        at org.elasticsearch.client.RestClient.convertResponse(RestClient.java:253)
        at org.elasticsearch.client.RestClient.access$900(RestClient.java:95)
        at org.elasticsearch.client.RestClient$1.completed(RestClient.java:298)
        ... 16 common frames omitted
```
## 解决办法
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html </br>

Automatic index creation is controlled by the action.auto_create_index setting. This setting defaults to true, which allows any index to be created automatically. You can modify this setting to explicitly allow or block automatic creation of indices that match specified patterns, or set it to false to disable automatic index creation entirely. Specify a comma-separated list of patterns you want to allow, or prefix each pattern with + or - to indicate whether it should be allowed or blocked. When a list is specified, the default behaviour is to disallow.
```
# Allow auto-creation of indices called twitter or index10, block the creation of indices that match the pattern index1*, and allow creation of any other indices that match the ind* pattern. Patterns are matched in the order specified.
PUT _cluster/settings{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*"
    }}
    
# Disable automatic index creation entirely.    
PUT _cluster/settings{
    "persistent": {
        "action.auto_create_index": "false"
    }}
    
# Allow automatic creation of any index. This is the default.    
PUT _cluster/settings{
    "persistent": {
        "action.auto_create_index": "true"
    }}
```
action.auto_create_index setting 在自建集群中默认是开启的，但在阿里云的 es 服务中默认没有开启，需在 kibana dev tools 开启执行以下命令开启。
```
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true"
    }
}
```
