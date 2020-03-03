# elk 集群_ansible playbook 部署(8)_xpack 用户认证配置
## Xpack 配置
logstash-elaticsearch-kibana 需配置 xpack 加密传输以及用户权限管理功能，证书仅需在 es 主节点生成，然后复制到各个节点。
### 证书配置 
生成 CA 证书  elastic-stack-ca.p12
```
# bin/elasticsearch-certutil ca 
ENTER ENTER 
```
es 主节点生成证书  elastic-certificates.p12
```
# bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
ENTER ENTER ENTER
```
es 主节点生成 p12 证书
```
bin/elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""
```
传输证书到 ansible 管理机并统一推送到 es 各个节点制定目录
```
---

- hosts: opd_es-02, opd_es-03
  tasks:
  - name: Copy Cert
    copy: src=roles/elasticsearch/files/cert/elastic-certificates.p12 dest=/sdata/usr/local/elasticsearch/config/elastic-certificates.p12 backup=yes force=no group=elk owner=elk mode=0600
```
### es 节点配置修改
所有 es 节点添加 elasticsearch.yml 配置
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```
使用 ansible 推送修改到 es 各节点，配置完成后重启 es 节点，所有节点配置 xpack 并重启完毕后才能配置集群密码，集群密码只需在 es 主节点生成即可
```
$ bin/elasticsearch-setup-passwords auto
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y
```

### kibana 配置修改
kibana 配置文件添加由es 生成的 kibana 用户及密码
```
# vim kibana.yml
elasticsearch.username: "kibana"
elasticsearch.password: "passwd"
```
配置完毕后重启 kibana

### logstash 配置修改
logstash 配置文件添加由 es 生成的 logstash 用户及密码
```
# vim logstash.yml
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.hosts: ["https://10.0.0.152:19200","https://10.0.0.153:19200","https://10.0.0.154:19200"]
xpack.monitoring.elasticsearch.username: "logstash_system"
xpack.monitoring.elasticsearch.password: "passwd"
```
修改 logstash 过滤配置添加 es 链接用户及密码
```
output {
  elasticsearch {
    hosts => ["10.0.0.152:19200","10.0.0.153:19200","10.0.0.154:19200"]
    index => "%{[type]}-log-%{+YYYY.MM.dd}"
    workers => 1
    user => "elastic"
    password => "passwd"
    #cacert => "/path/to/ca.crt"
  }
}
```
配置完毕后重启 logstash

### Xpack 验证
使用 elastic 用户登录 kibana，查看账户权限功能是否已启动，es 各节点是否通信正常。
