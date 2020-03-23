# docker 部署 zabbix server 更新开启 smnp 监控华为交换机
## zabbix-server 容器内部 snmp 配置
### 停用原有容器
```
docker stop cb2acde54ddd
```
### 修改容器启动脚本
将 /usr/share/snmp/mibs 等几个snmp相关持久化，映射端口 161 到宿主机。
注意：不持久化  /usr/share/snmp/mibs 会导致安装 snmpwalk 的时出现空间不足的问题。
```
# cd /sdata/software/docker/script/Docker-run
# vim zabbix-server-4.0.1-02.sh
#! /bin/bash

docker run --name zabbix-server-4.0.1-02 -t \
  -p 161:161 \
  -v /sdata/software/docker/data/zabbix-server/enc:/var/lib/zabbix/enc \
  -v /sdata/software/docker/data/zabbix-server/snmp/mibs:/usr/share/snmp/mibs \
  -v /sdata/software/docker/data/zabbix-server/modules:/var/lib/zabbix/modules \
  -v /sdata/software/docker/data/zabbix-server/snmptraps:/var/lib/zabbix/snmptraps \
  -p 10051:10051 \
  --hostname zabbix-server \
  -e DB_SERVER_HOST="mysql-5.6-01" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="pwd"\
  -e MYSQL_ROOT_PASSWORD="pwd" \
  -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
  --link mysql-5.6-01:mysql \
  --link zabbix-java-gateway-4.0.1-01:zabbix-java-gateway \
  --link zabbix-snmptraps-4.0.1-01:zabbix-snmptraps \
  --volumes-from zabbix-snmptraps-4.0.1-01 \
  -v /sdata/software/docker/data/zabbix-server/alertscripts:/usr/lib/zabbix/alertscripts \
  -v /sdata/software/docker/data/zabbix-server/externalscripts:/usr/lib/zabbix/externalscripts \
  -d zabbix-server-4.0.1
```
### 启动容器
```
# sh zabbix-server-4.0.1-02.sh
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                            NAMES
2e1506b5e8c0        zabbix-server-4.0.1         "docker-entrypoint.sh"   3 hours ago         Up 3 hours          0.0.0.0:161->161/tcp, 0.0.0.0:10051->10051/tcp   zabbix-server-4.0.1-02
3729e918c67d        zabbix-web-4.0.1            "docker-entrypoint.sh"   10 days ago         Up 10 days          0.0.0.0:80->80/tcp, 443/tcp                      zabbix-web-4.0.1-01
e34f0a76afb6        zabbix-snmptraps-4.0.1      "/usr/bin/supervisor…"   10 days ago         Up 10 days          0.0.0.0:162->162/udp                             zabbix-snmptraps-4.0.1-01
c4196481196a        zabbix-java-gateway-4.0.1   "docker-entrypoint.sh"   10 days ago         Up 10 days          0.0.0.0:10052->10052/tcp                         zabbix-java-gateway-4.0.1-01
89baa5ab3918        mysql-centos-5.6            "docker-entrypoint.s…"   11 days ago         Up 11 days          0.0.0.0:3306->3306/tcp                           mysql-5.6-01
```
### 进入 zabbix-server 新容器安装 snmpwalk
```
# docker exec -it 2e1506b5e8c0 /bin/bash
# yum install net-snmp-utils
```
### 容器内 snmp 测试
```
# snmpwalk -v 2c -c 团体名 173.16.249.233
# snmpwalk -v 2c -c 团体名 173.16.249.233 "1.3.6.1.4.1.2011.5.25.31.1.1.1.1.5.67108873"
SNMPv2-SMI::enterprises.2011.5.25.31.1.1.1.1.5.67108873 = INTEGER: 12
```
