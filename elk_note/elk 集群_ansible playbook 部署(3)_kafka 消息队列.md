# elk 集群_ansible playbook 部署(3)_kafka 消息队列
生产 filebeat 节点位于阿里云生产环境内，而 logstash 及 es 集群使用本地办公网主机资源，中间经过较长的公网传输。
为防止数据丢失、安全性、性能考虑，需在 filebeat 及 logstash 间加一层 kafka 消息缓存，且需要公网加密传输(减少vpn带宽消耗)。


