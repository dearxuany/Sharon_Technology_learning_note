# elk 集群_ansible playbook 部署(5)_elasticsearch 集群
此处搭建 elasticsearch 三节点集群，三节点均可选举为 master，均接受请求及存储数据。
```
# elk 集群 ansible inventory
[opd_ansible-01]
127.0.0.1

[opd_logstash-01]
10.0.0.151
[opd_logstash-01:vars]
server_names="opd-elk-01"

[opd_es-01]
10.0.0.152
[opd_es-01:vars]
server_names="opd-elk-02"

[opd_es-02]
10.0.0.153
[opd_es-02:vars]
server_names="opd-elk-03"

[opd_es-03]
10.0.0.154
[opd_es-03:vars]
server_names="opd-elk-04"

[opd_kibana-01]
10.0.0.151
[opd_kibana-01:vars]
server_names="opd-elk-01"
```
每个 es 节点都需要安装 java 8 或以上版本 java，需要添加 elk 用户作为 es 进程管理用户，此处使用 ansible 统一配置完毕。

## es 初始化安装
下载 elasticsearch 7.1.1 版本到 ansible 服务端主机
```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.1.1-linux-x86_64.tar.gz
```
elasticsearch 启动脚本
```
#! /bin/bash
nohup /sdata/usr/local/elasticsearch/bin/elasticsearch >> /sdata/var/log/elasticsearch/elasticsearch-start.log 2>&1 &
```
elasticsearch 初始化安装 playbook</br>
注意：启动 es 节点需要调整 linux 系统参数，如虚拟内存大小、可以打开的文件数量、禁止使用 swap
```
---

- hosts: opd_es-01, opd_es-02, opd_es-03
  tasks:
  - name: Copy Package
    copy: src=files/elasticsearch-7.1.1-linux-x86_64.tar.gz dest=/sdata/usr/local/src/elasticsearch.tar.gz
  - name: Tar Package
    shell: tar -xvz -f /sdata/usr/local/src/elasticsearch.tar.gz -C /sdata/usr/local
  - name: Rename elasticsearch
    shell: mv /sdata/usr/local/elasticsearch-7.1.1 /sdata/usr/local/elasticsearch
  - name: Copy Elasticsearch Startup Script
    copy: src=files/start_elasticsearsh.sh dest=/sdata/usr/local/elasticsearch/start_elasticsearsh.sh
  - name: open user lock
    become: yes
    become_user: root
    shell: chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
  - name: Create elk User
    become: yes
    become_user: root
    user: name=elk state=present createhome=no shell=/bin/bash
  - name: shutdown user lock
    become: yes
    become_user: root
    shell: chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow
  - name: Modify Elasticsearch Basedir Permission
    file: path=/sdata/usr/local/elasticsearch owner=elk group=elk recurse=yes
  - name: Create Elasticsearch Start Log Dir
    file: path=/sdata/var/log/elasticsearch state=directory owner=elk group=elk mode=0755
  - name: Create Elasitcsearch Start Log
    file: path=/sdata/var/log/elasticsearch/elasticsearch-start.log state=touch owner=elk group=elk mode=0644
  - name: Create Elasticsearch Data Dir
    file: path=/sdata/data/elasticsearch state=directory owner=elk group=elk mode=0755
  - name: set the number of open file handles
    become: yes
    become_user: root
    lineinfile:
      dest: /etc/security/limits.conf
      insertbefore: '{{ item.insertbefore }}'
      line: '{{ item.line }}'
    with_items:
      - { insertbefore: '^# End of file', line: 'elk soft nofile 65536'}
      - { insertbefore: '^# End of file', line: 'elk hard nofile 65536'}
      - { insertbefore: '^# End of file', line: 'elk soft memlock unlimited'}
      - { insertbefore: '^# End of file', line: 'elk hard memlock unlimited'}
  - name: increase max virtual memory areas
    become: yes
    become_user: root
    lineinfile:
      dest: /etc/sysctl.conf
      insertafter: '{{ item.insertafter }}'
      line: '{{ item.line }}'
    with_items:
      - { insertafter: '^# For more information', line: 'vm.max_map_count=262144'}
      - { insertafter: '^# For more information', line: 'vm.swappiness=0'}
  - name: update the  max virtual memory areas
    shell: sudo sysctl -p

```
elasticsearch 初始化 playbook 执行
```
#! /bin/bash
ansible-playbook -i /sdata/app/ansible-deploy/alihn1-playbook/inventory/elk_hosts --user=ops --private-key=/sdata/snail/.ssh/id_rsa -s /sdata/app/ansible-deploy/alihn1-playbook/roles/elasticsearch/main.yml
```
登录到 es 节点，启动 elasticsearch
```
su - elk
sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```
验证初始化节点可用性
```
$ curl http://127.0.0.1:9200
{
  "name" : "gzyw53-opd-elk-02",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "mUONBs_3RNO4TKvzu21Fjg",
  "version" : {
    "number" : "7.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "7a013de",
    "build_date" : "2019-05-23T14:04:00.380842Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## elasticsearch 配置
es 配置部署 ansible playbook
```
---

- hosts: opd_ansible-01
  tasks:
  - name: git pull elasticsearch hosts config
    become: yes
    become_user: snail
    shell: cd /sdata/app/ansible-deploy/alihn1-playbook/roles/elasticsearch/files/elk-elasticsearch-conf && git pull origin prd

- hosts: prd_wikies-01
  tasks:
  - name: Copy elasticsearch config to the node
    copy: src=files/elk-elasticsearch-conf/{{ server_names }}-config/elasticsearch.yml dest=/sdata/usr/local/elasticsearch/config/elasticsearch.yml backup=yes force=yes group=elk owner=elk mode=0644
  - name: Copy jvm config to the node
    copy: src=files/elk-elasticsearch-conf/{{ server_names }}-config/jvm.options dest=/sdata/usr/local/elasticsearch/config/jvm.options backup=yes force=yes group=elk owner=elk mode=0644
```
ansible playbook 推送配置文件
```
#! /bin/bash
ansible-playbook -i /sdata/app/ansible-deploy/alihn1-playbook/inventory/elk_hosts --user=ops --private-key=/sdata/snail/.ssh/id_rsa -s /sdata/app/ansible-deploy/alihn1-playbook/roles/elasticsearch/changeConfig.yml
```

## es 节点配置
### 主节点配置
elasticsearch.yml 修改集群信息，注意集群名不能用默认的，否则网段内任何使用默认集群名的 es 节点都会被 es 自动加入集群中。
```
# 集群名
cluster.name: cluster-elasticsearch-01
# 节点名
node.name: gzyw53-elasticsearch-01
path.data: /sdata/data/elasticsearch
path.logs: /sdata/var/log/elasticsearch
# 禁止使用 swapping 分区（会影响 es 性能）
bootstrap.memory_lock: true
# 节点ip及监听端口
network.host: 10.0.0.152
http.port: 19200
# es 节点间交互端口
transport.tcp.port: 19300
# 集群节点发现
discovery.seed_hosts: ["10.0.0.152:19300", "10.0.0.153:19300","10.0.0.154:19300"]
# 初始主节点配置
cluster.initial_master_nodes: ["gzyw53-elasticsearch-01"]
```
jvm.options 修改 JVM 堆内存
```
-Xms16g
-Xmx16g
```
启动 es 主节点
```
su - elk
sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```
验证 es 主节点
```
# ps -ef|grep elasticsearch
elk        366     1 16 18:21 pts/1    00:00:59 /sdata/usr/local/jdk/bin/java -Xms16g -Xmx16g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-108635704526937116 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Dio.netty.allocator.type=pooled -Des.path.home=/sdata/usr/local/elasticsearch -Des.path.conf=/sdata/usr/local/elasticsearch/config -Des.distribution.flavor=default -Des.distribution.type=tar -Des.bundled_jdk=true -cp /sdata/usr/local/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch
# netstat -tunpl|grep 19200
tcp6       0      0 10.0.0.152:19200        :::*                    LISTEN      366/java
# netstat -tunpl|grep 19300
tcp6       0      0 10.0.0.152:19300        :::*                    LISTEN      366/java  
```
内网浏览器上访问 10.0.0.152:19200
```
{
  "name" : "gzyw53-elasticsearch-01",
  "cluster_name" : "cluster-elasticsearch-01",
  "cluster_uuid" : "b0u2IJIERfu29L8-lQsFKg",
  "version" : {
    "number" : "7.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "7a013de",
    "build_date" : "2019-05-23T14:04:00.380842Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### es 节点2配置
elasticsearch.yml 修改集群信息
```
cluster.name: cluster-elasticsearch-01
node.name: gzyw53-elasticsearch-02
path.data: /sdata/data/elasticsearch
path.logs: /sdata/var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 10.0.0.153
http.port: 19200
transport.tcp.port: 19300
discovery.seed_hosts: ["10.0.0.152:19300", "10.0.0.153:19300","10.0.0.154:19300"]
cluster.initial_master_nodes: ["gzyw53-elasticsearch-01"]
```
jvm.options 修改 JVM 堆内存
```
-Xms8g
-Xmx8g
```
启动 es 节点2
```
su - elk
sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```
查看 es 节点2 启动状况
```
$ curl http://10.0.0.153:19200
{
  "name" : "gzyw53-elasticsearch-02",
  "cluster_name" : "cluster-elasticsearch-01",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "7a013de",
    "build_date" : "2019-05-23T14:04:00.380842Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
### es 节点3配置
elasticsearch.yml 修改集群信息
```
cluster.name: cluster-elasticsearch-01
node.name: gzyw53-elasticsearch-03
path.data: /sdata/data/elasticsearch
path.logs: /sdata/var/log/elasticsearch
bootstrap.memory_lock: true
network.host: 10.0.0.154
http.port: 19200
transport.tcp.port: 19300
discovery.seed_hosts: ["10.0.0.152:19300", "10.0.0.153:19300", "10.0.0.154:19300"]
# 可获得 master 资格节点个数 N/2+1
discovery.zen.minimum_master_nodes: 2
# 节点发现超时时间
discovery.zen.ping_timeout: 120s
# 初始化主节点
cluster.initial_master_nodes: ["gzyw53-elasticsearch-01"]
```
jvm.options 修改 JVM 堆内存
```
-Xms12g
-Xmx12g
```
启动 es 节点3
```
su - elk
sh /sdata/usr/local/elasticsearch/start_elasticsearsh.sh
```
查看 es 节点3 启动状况
```
$ curl http://10.0.0.154:19200
{
  "name" : "cluster-elasticsearch-03",
  "cluster_name" : "insnail-elasticsearch-01",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "7a013de",
    "build_date" : "2019-05-23T14:04:00.380842Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## es 节点防火墙配置
查看系统防火墙状态
```
# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 一 2019-09-30 10:04:16 CST; 3s ago
```
查看 iptables 配置策略
```
# iptables -L
```
es 各个节点开启 19200，允许 10.0.0.151 即 logstash 及 kibana 所在主机访问
```
# firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.0.0.151" port protocol="tcp" port="19200" accept"
```
es 各个节点开启 19300，允许 es 其余节点访问，用于 es 节点间通信
```
# firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.0.0.153" port protocol="tcp" port="19300" accept"
# firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.0.0.154" port protocol="tcp" port="19300" accept"
```
配置完毕后重新加载防火墙
```
# firewall-cmd --reload
```
验证配置
```
# iptables -L
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  10.0.0.151           anywhere             tcp dpt:19200 ctstate NEW
ACCEPT     tcp  --  10.0.0.153           anywhere             tcp dpt:19300 ctstate NEW
ACCEPT     tcp  --  10.0.0.154           anywhere             tcp dpt:19300 ctstate NEW
```
各主机配置完毕后，相互使用 telnet 测试端口是否能使用正常，同时保证除安全策略允许外的主机无法访问端口。

