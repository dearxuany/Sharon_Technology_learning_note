# elk 集群_ansible playbook 部署(3)_logstash
## logstash 安装
```
# curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.1.1.tar.gz
# tar -xvz -f /sdata/usr/local/logstash/logstash-7.1.1.tar.gz -C /sdata/usr/local/
# mv /sdata/usr/local/logstash-7.1.1 /sdata/usr/local/logstash
# mkdir -p /sdata/data/logstash
# mkdir -p /sdata/var/log/logstash
# chown -R elk:elk /sdata/usr/local/logstash
# chown -R elk:elk /sdata/data/logstash
# chown -R elk:elk /sdata/var/log/logstash
```
检验 logstash 安装包完整性
```
-bash-4.2$ ./bin/logstash -e 'input { stdin { } } output { stdout {} }'
Sending Logstash logs to /sdata/usr/local/logstash/logs which is now configured via log4j2.properties
[2019-09-16T23:50:03,143][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2019-09-16T23:50:03,166][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.1.1"}
[2019-09-16T23:50:10,472][INFO ][logstash.javapipeline    ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>500, :thread=>"#<Thread:0x7fe4efc2 run>"}
[2019-09-16T23:50:10,567][INFO ][logstash.javapipeline    ] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2019-09-16T23:50:10,669][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2019-09-16T23:50:10,954][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
sss
/sdata/usr/local/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
{
    "@timestamp" => 2019-09-17T03:50:16.451Z,
          "host" => "gzyw53-opd-elk-01",
      "@version" => "1",
       "message" => "sss"
}
```
## logstash 配置
logstash 基础配置
```
# vim /sdata/usr/local/logstash/config/logstash.yml
node.name: gzyw53-logstash-01
path.data: /sdata/data/logstash
log.level: info
path.logs: /sdata/var/log/logstash
```
logstash jvm 配置
```
# vim /sdata/usr/local/logstash/config/jvm.options
-Xms4g
-Xmx4g
```
logstash 链接配置
```
# 简单配置测试用，未开启加密和过滤

# mkdir /sdata/usr/local/logstash/config/conf.d
# vim /sdata/usr/local/logstash/config/conf.d/gzyw53-logstash-01.conf
input {
  kafka {
    bootstrap_servers => "alihn1-opd-elk-01.snail:19092"
    topics => ["kafkatest"]
    codec => "json"
    decorate_events => true
    type => "kafkatest"
  }


output {
  elasticsearch {
    hosts => ["10.0.0.152:19200"]
    index => "%{[type]}-log-%{+YYYY.MM.dd}"
    workers => 1
    #user => "elastic"
    #password => "passwd"
    #cacert => "/path/to/ca.crt"
  }
}
```
logstash 主机 /etc/hosts 配置
```
vim /etc/hosts
1.2.3.4  alihn1-opd-elk-01.snail
```
注意：填写阿里云 kafka 主机或 NAT 网关公网 ip ，主机名设置必须和 kafka 阿里云主机的主机名一样。

## logstash 启动
启动命令
```
# vim /sdata/usr/local/logstash/start_logstash.sh

#! /bin/bash
nohup /sdata/usr/local/logstash/bin/logstash -f /sdata/usr/local/logstash/config/conf.d/logstash.conf >> /sdata/var/log/logstash/logstash-start.log 2>&1 &
```
## kafka - logstash 公网链接测试
使用kafka官方测试脚本测试
### 查看 kafka 主题
在本地网络 logstash 主机（10.0.0.151）链接阿里云 kafka 主机
```
# ./bin/kafka-topics.sh --zookeeper alihn1-opd-elk-01:2181 --list
__consumer_offsets
kafkatest
```
### 生产者测试
在阿里云主机（172.17.0.1）生产数据发送给阿里云 kafka 主机
```
# ./bin/kafka-console-producer.sh --broker-list alihn1-opd-elk-01:19092 --topic kafkatest
>aaa
>ddd
>vvv
```

### 消费者测试
在本地网络 logstash 主机（10.0.0.151）消费阿里云 kafka 主机 kafka 主题的数据
```
# ./bin/kafka-console-consumer.sh --bootstrap-server  alihn1-opd-elk-01:19092 --topic kafkatest --from-beginning
aaa
ddd
vvv
```
