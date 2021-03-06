# elk 集群_ansible playbook 部署(2)_filebeat
因为 filebeat 是作为 agent 分布在各个主机，配置分散。为方便统一管理，将各主机的 filebeat 的配置文件托管到 gitlab 上，jenkins 触发部署 filebeat 时从 gitlab 拉取最新的配置到 ansible 所在主机并发送到目标主机。 filebeat 启动需依赖后部的 kafka，需 kafka 启动成功后才能成功启动，建议部署完 kafka 后再统一部署 filebeat，避免多次修改 filebeat 配置文件。
## ansible 虚拟环境激活
启动阿里云 vpc 中的 ansible 服务
```
source /sdata/snail/.py3-a2.5-env/bin/activate
source /sdata/snail/.py3-a2.5-env/ansible/hacking/env-setup -q
```
## filebeat ansible playbook
filebeat ansible 部署 playbook
```
---


- hosts: opd_ansible-01
  tasks:
  - name: git pull filebeat hosts config
    become: yes
    become_user: snail
    shell: cd /sdata/app/ansible-deploy/alihn1-playbook/roles/filebeat/files/elk-filebeat-conf && git pull origin prd


- hosts: prd_streaming-01
  tasks:
  - name: Copy Package
    copy: src=roles/filebeat/files/filebeat.tar.gz  dest=/sdata/usr/local/src/filebeat.tar.gz
  - name: Tar Packeage
    shell: cd /sdata/usr/local; tar -xvz -f /sdata/usr/local/src/filebeat.tar.gz -C /sdata/usr/local
  - name: Copy Filebeat Config
    copy: src=roles/filebeat/files/elk-filebeat-conf/{{ server_names }}-filebeat.yml dest=/sdata/usr/local/filebeat/filebeat.yml
  - name: Copyt Filebeat Startup Script
    copy: src=roles/filebeat/files/start_filebeat.sh dest=/sdata/usr/local/filebeat/start_filebeat.sh
  - name: Copy Filebeat Startup File To Systemd
    copy: src=roles/filebeat/files/filebeat.service dest=/etc/systemd/system/filebeat.service
  - name: Reload Filebeat Systemd Unit File
    shell: systemctl daemon-reload
  - name: open user lock
    become: yes
    become_user: root
    shell: chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
  - name: Create Filebeat User
    become: yes
    become_user: root
    user: name=elk state=present createhome=no shell=/usr/sbin/nologin
  - name: shutdown user lock
    become: yes
    become_user: root
    shell: chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow
  - name: Modify Filebeat Basedir Permission
    file: path=/sdata/usr/local/filebeat owner=elk group=elk recurse=yes
  - name: Create Filebeat Start Log Dir
    file: path=/sdata/var/log/filebeat state=directory owner=elk group=elk mode=0755
  - name: Start Filebeat With System Start
    shell: systemctl enable filebeat
  - name: Create Filebeat Start Log
    file: path=/sdata/var/log/filebeat/filebeat-start.log state=touch owner=elk group=elk mode=0644
  - name: Run Filebeat by ELK User
    shell: systemctl restart filebeat

```
filebeat 配置变更 ansible playbook
```
---


- hosts: opd_ansible-01
  tasks:
  - name: git pull filebeat hosts config
    become: yes
    become_user: snail
    shell: cd /sdata/app/ansible-deploy/alihn1-playbook/roles/filebeat/files/elk-filebeat-conf && git pull origin prd


- hosts: prd_streaming-01
  tasks:
  - name: stop filebeat
    shell: systemctl stop filebeat
  - name: copy filebeat config
    copy: src=roles/filebeat/files/elk-filebeat-conf/{{ server_names }}-filebeat.yml dest=/sdata/usr/local/filebeat/filebeat.yml
  - name: Run Filebeat by ELK User
    shell: systemctl restart filebeat
```
## ansible inventory
将要部署 filebeat 的主机的主机名写入 ansible 的 inventory 中并将主机名作为变量传入到 playbook 中
```
# vim /sdata/app/ansible-deploy/alihn1-playbook/inventory/elk_hosts

[prd_ansible-01]
127.0.0.1

[prd_nginx-01]
alihn1-prd-nginx-01
[prd_nginx-01:vars]
server_names="alihn1-prd-nginx-01"

[prd_nginx-02]
alihn1-prd-nginx-02
[prd_nginx-02:vars]
server_names="alihn1-prd-nginx-02"

[prd_nginx-03]
alihn1-prd-nginx-03
[prd_nginx-03:vars]
server_names="alihn1-prd-nginx-03"

[opd_elk-01]
alihn1-opd-elk-01
[opd_elk-01:vars]
server_names="alihn1-opd-elk-01"

[opd_jenkins-01]
alihn1-opd-jenkins-01
[opd_jenkins-01:vars]
server_names="alihn1-opd-jenkins-01"
```
## filebeat 配置
进入 filebeat 配置文件存储目录，新建一份以目标主机名命名的 filebeat 配置文件
```
cd /sdata/app/ansible-deploy/alihn1-playbook/roles/filebeat/files
cp ./filebeat.yml ./alihn1-opd-jenkins-01-filebeat.yml
```
修改 filebeat 配置，注意不同阿里云主机使用不同配置
```
# vim ./alihn1-opd-jenkins-01-filebeat.yml

filebeat.inputs:
- type: log
  enabled: true
  paths:
  - /sdata/app/gitAnalysis/results/analysisResults.txt
  fields:
    log_source: opd-gitAnalysis-01
    log_topics: opdData01
  tags: ["alihn1","opd","gitAnalysis"]
  
- type: log
  enabled: true
  paths:
  - /sdata/data/connection_monitor/urlInfo.txt
  - /sdata/data/connection_monitor/urlStatistic.txt
  fields:
    log_source: opd-monitor-01
    log_topics: opdMonitor01
  tags: ["alihn1","opd","monitor","urlCheck","data"]
  
- type: log
  enabled: true
  paths:
  - /sdata/var/log/connection_monitor/urlRequest.log
  fields:
    log_source: opd-monitor-01
    log_topics: opdMonitor01
  tags: ["alihn1","opd","monitor","urlCheck","log","error"]
    
output.kafka:
  hosts: ["alihn1-opd-elk-01:9092"]
  topic: '%{[fields][log_topics]}'
  partition.round_robin:
    reachable_only: false
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000

```
systemctl filebeat unit 文件
```
# cat filebeat.service
[Unit]
Description=filebeat
After=network.target
[Service]
Type=forking
User=elk
Group=elk
ExecStart=/bin/sh /sdata/usr/local/filebeat/start_filebeat.sh
Restart=always
RestartSec=1
[Install]
WantedBy=multi-user.target

```
systemctl filebeat 启动脚本
```
# cat start_filebeat.sh
#!/bin/sh
nohup /sdata/usr/local/filebeat/filebeat -e -c /sdata/usr/local/filebeat/filebeat.yml >> /sdata/var/log/filebeat/filebeat-start.log 2>&1 &
```
## filebeat 部署安装 
执行 ansible playbook 安装 filebeat，注意需要 kafka 已完成启动可监听 9092 后才能正常启动 filebeat。
```
#! /bin/bash

ansible-playbook -i /sdata/app/ansible-deploy/alihn1-playbook/inventory/elk_hosts --user=ops --private-key=/sdata/snail/.ssh/id_rsa -s /sdata/app/ansible-deploy/alihn1-playbook/roles/filebeat/main.yml
```
filebeat 启动验证
```
# ps -ef|grep filebeat
elk      27254     1  0 10:07 ?        00:00:00 /sdata/usr/local/filebeat/filebeat -e -c /sdata/usr/local/filebeat/filebeat.yml
```
## jenkins 一键部署 filebeat
jenkins 配置 shell 执行本地主机脚本</br>
注：ansible 和 jenkins 需要在同一台主机，否则需要通过 SSH 操作</br>
```
cd /sdata/app/ansible-deploy && git pull origin prd
. /sdata/app/ansible-deploy/alihn1-playbook/venvActivate.sh
sh /sdata/app/ansible-deploy/alihn1-playbook/roles/filebeat/deployScript/deployFilebeat.sh
```


