# elk 集群_ansible playbook 部署(6)_kibana 可视化.md
## kibana 初始化安装
kibana 下载
```
# wget https://artifacts.elastic.co/downloads/kibana/kibana-7.1.1-linux-x86_64.tar.gz
```
kibana 启动脚本
```
# vim start_kibana.sh
#! /bin/bash
nohup  /sdata/usr/local/kibana/bin/kibana >> /sdata/var/log/kibana/kibana-start.log 2>&1 &
```
kibana 初始化安装 ansible playbook
```
---

- hosts: opd_kibana-01
  tasks:
  - name: Copy Package
    copy: src=files/kibana-7.1.1-linux-x86_64.tar.gz dest=/sdata/usr/local/src/kibana.tar.gz
  - name: Tar Package
    shell: tar -xvz -f /sdata/usr/local/src/kibana.tar.gz -C /sdata/usr/local
  - name: Rename kibana
    shell: mv /sdata/usr/local/kibana-7.1.1-linux-x86_64 /sdata/usr/local/kibana
  - name: Copy Kibana Startup Script
    copy: src=files/start_kibana.sh dest=/sdata/usr/local/kibana/start_kibana.sh
  - name: Modify kibana Basedir Permission
    file: path=/sdata/usr/local/kibana owner=elk group=elk recurse=yes
  - name: Create kibana Start Log Dir
    file: path=/sdata/var/log/kibana state=directory owner=elk group=elk mode=0755
  - name: Create kibana Start Log
    file: path=/sdata/var/log/kibana/kibana-start.log state=touch owner=elk group=elk mode=0644
```

ansible kibana 推送部署
```
#! /bin/bash
ansible-playbook -i /sdata/app/ansible-deploy/alihn1-playbook/inventory/elk_hosts --user=ops --private-key=/sdata/snail/.ssh/id_rsa -s /sdata/app/ansible-deploy/alihn1-playbook/roles/kibana/main.yml
```
## kibana 配置
由于 kibana 为单节点此处不使用 ansible 推送配置文件，直接登录 kibana 节点进行配置
```
# vim /sdata/usr/local/kibana/config/kibana.yml
server.port: 5601
server.host: "10.0.0.151"
server.name: "gzyw53-kibana-01"
elasticsearch.hosts: ["http://10.0.0.152:19200"]
```
启动 kibana，注意需要先启动 elasticsearch 才能启动 kibana
```
su - elk
sh /sdata/usr/local/kibana/start_kibana.sh
```
验证 kibana 初始化安装
```
# netstat -tunpl|grep 5601
tcp        0      0 10.0.0.151:5601         0.0.0.0:*               LISTEN      3501/node  
```

## kibana 页面验证
内网浏览器登录 kibana 
```
http://10.0.0.151:5601
```
kibana 上打开 elk 集群监控，注意此时集群状态是绿色的，主分片有5个，副分片也是5个。

## kibana 域名配置
### nginx 反向代理
域名配置文件，http 80 端口需跳转到 https 443
```
# vim /data/software/nginx/conf/conf.d/elk.domainname.com.conf

server {
    listen 80;
    server_name elk.domainname.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}

server {
    #listen 80;
    listen 443 ssl;
    server_name elk.domainname.com;

    access_log /data2/log/nginx/elk.domainname.com_access.log;
    error_log /data2/log/nginx/elk.domainname.com_error.log notice;

    ssl_protocols       SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_certificate     cert/domainname.com.pem;
    ssl_certificate_key cert/domainname.com.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;
      
    location / {
            proxy_pass http://10.0.0.151:5601;
            #proxy_http_version 1.1;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_connect_timeout 600;
            proxy_read_timeout 600;
            proxy_send_timeout 600;
            proxy_cache_bypass $http_upgrade;
        }
}
```
nginx 配置检查
```
nginx -t -c /sdata/etc/nginx/nginx.conf
```
nginx 配置重新加载
```
nginx -s reload
```
### DNS 配置
本地 DNS 配置域名解析到 nginx 虚 ip 10.0.0.4，阿里云域名添加域名解析到本地网络入口网关 ip
```
 vim /etc/dnsmasq.d/domainname.com.conf
 
 address=/elk.domainname.com/10.0.0.4
```
重启dnsmasq
```
systemctl restart dnsmasq
```
检查域名解析地址
```
C:\Users\IT>nslookup elk.domainname.com
服务器:  ns1.domainname.com
Address:  10.0.0.51

名称:    kvm.domainname.com
Address:  10.0.0.4
Aliases:  elk.domainname.com
```
