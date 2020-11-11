# docker 部署 zabbix server
## 依赖
### 软件依赖
docker-ce 18.09.6
### 用户设置
新建 zabbix及mysql 用户并添加到docker组
```
#useradd zabbix
#cat /etc/group|grep docker
docker:x:994:snail
# usermod -aG docker zabbix
# useradd mysql
# usermod -aG docker mysql
# cat /etc/group|grep docker
docker:x:994:snail,zabbix,mysql
```
注意：需重启docker才生效

## 镜像拉取
使用 zabbix 官方的镜像  https://www.zabbix.com/download_docker 、mysql 镜像 https://docs.docker.com/samples/library/mysql/

### 查找仓库官方镜像
mysql 镜像
```
docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   8448                [OK]           
```
zabbix-java-gateway 镜像
```
docker search zabbix-java-gateway
NAME                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zabbix/zabbix-java-gateway                Zabbix Java Gateway                             16                                      [OK]
```
zabbix-snmptraps 镜像
```
docker search zabbix-snmptraps
NAME                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zabbix/zabbix-snmptraps                Receiving SNMP traps to Zabbix server or Zab…   11                                      [OK]
```
zabbix-server 镜像
```
docker search zabbix-server
NAME                                       DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zabbix/zabbix-server-mysql                 Zabbix Server with MySQL database support       201                                     [OK]
```
zabbix-web 镜像
```
docker search zabbix-web
NAME                                            DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zabbix/zabbix-web-nginx-mysql                   Zabbix frontend based on Nginx web-server wi…   109                                     [OK]
```
zabbix-agent 镜像
```
docker search zabbix-agent
NAME                                        DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zabbix/zabbix-agent                         Zabbix agent with TLS encryption support        112                                     [OK]
```
### 拉取官方镜像
```
# docker pull mysql:5.6
# docker pull zabbix/zabbix-java-gateway:centos-4.0.1
# docker pull zabbix/zabbix-snmptraps:centos-4.0.1
# docker pull zabbix/zabbix-server-mysql:centos-4.0.1
# docker pull zabbix/zabbix-web-nginx-mysql:centos-4.0.1
# docker pull zabbix/zabbix-agent:centos-4.0.1
```
```
# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
mysql                           5.6                 7b01f1418bd7        9 days ago          256MB
zabbix/zabbix-agent             centos-4.0.1        d54549f25162        9 months ago        224MB
zabbix/zabbix-snmptraps         centos-4.0.1        ce94715002cf        9 months ago        276MB
zabbix/zabbix-web-nginx-mysql   centos-4.0.1        d3dc40d2e097        9 months ago        385MB
zabbix/zabbix-java-gateway      centos-4.0.1        68a52a123c49        9 months ago        359MB
zabbix/zabbix-server-mysql      centos-4.0.1        8f5de9266d28        9 months ago        327MB
```
## 容器创建启动
### 宿主机创建数据持久化目录
```
# mkdir -p /sdata/software/docker/data/mysql
# mkdir /sdata/software/docker/data/zabbix-java-gateway
# mkdir /sdata/software/docker/data/zabbix-snmptraps
# mkdir /sdata/software/docker/data/zabbix-server
# mkdir /sdata/software/docker/data/zabbix-web
# mkdir /sdata/software/docker/data/zabbix-agent
```
### mysql 安装配置
#### 编写 Dockerfile 修改时区及编码并构建新镜像
```
# su mysql
# cd /sdata/software/docker/data/mysql
$ vim Dockerfile
FROM mysql:5.6
ENV LANG en_US.utf8
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone \
$ docker build -t mysql-centos-5.6 .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM mysql:5.6
---> 7b01f1418bd7
Step 2/3 : ENV LANG en_US.utf8
---> Running in 4a2a7cb0c5f8
Removing intermediate container 4a2a7cb0c5f8
---> 5542063464ad
Step 3/3 : RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&     echo Asia/Shanghai > /etc/timezone
---> Running in 0a37c905ceff
Removing intermediate container 0a37c905ceff
---> fffbf466f6c4
Successfully built fffbf466f6c4
Successfully tagged mysql-centos-5.6:latest
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
mysql-centos-5.6                latest              fffbf466f6c4        24 seconds ago      256MB
```
#### 配置参数并启动 mysql 容器
容器启动脚本、参数
```
$ mkdir -p /sdata/software/docker/script/Docker-run
$ cd /sdata/software/docker/script/Docker-run
$ vim mysql-5.6-01.sh
#! /bin/bash

docker run --hostname mysql --name mysql-5.6-01 -t \
-p 3306:3306 \
-e MYSQL_USER="zabbix" \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_PASSWORD="passwd" \
-e MYSQL_ROOT_PASSWORD="passwd" \
-v /sdata/software/docker/data/mysql:/var/lib/mysql:rw \
-d mysql-centos-5.6
```
启动容器
```
$ sh mysql-5.6-01.sh
89baa5ab3918a565f87a18ad245eb5edd234d12f5c6d7280d57550e61775841d
```
查看容器启动状态
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
89baa5ab3918        mysql-centos-5.6    "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp   mysql-5.6-01
```
#### mysql 验证
```
$ netstat -tunlp|grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      -
$ ps -ef|grep mysql
polkitd  15939 15921  0 02:46 pts/0    00:00:00 mysqld
# mysql -h10.0.0.138 -uzabbix -p
Enter password:
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| zabbix             |
+--------------------+
2 rows in set (0.00 sec)

````
### zabbix-java-gateway 安装配置
#### 编写 Dockerfile 修改时区及编码并构建新镜像
```
# cd /sdata/software/docker/data/zabbix-java-gateway
# su zabbix
```
dockerfile 编辑
```
$ vim Dockerfile
FROM zabbix/zabbix-java-gateway:centos-4.0.1
ENV LANG en_US.utf8
RUN yum install -y tzdata
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone \
```
使用 dockerfile 构建镜像
```
$ docker build -t zabbix-java-gateway-4.0.1 .
```
查看镜像
```
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
zabbix-java-gateway-4.0.1       latest              5efe24714a5a        4 minutes ago       472MB
```
#### 配置参数并启动 zabbix-java-gateway 容器
容器启动脚本
```
$ cd /sdata/software/docker/script/Docker-run
$ vim zabbix-java-gateway-4.0.1-01.sh
#! /bin/bash

docker run --hostname zabbix-java-gateway \
--name zabbix-java-gateway-4.0.1-01 -t \
-p 10052:10052 \
-d zabbix-java-gateway-4.0.1
```
运行容器
```
$ sh zabbix-java-gateway-4.0.1-01.sh
c4196481196a5f091a7e77b31a5957e70a464d4f531dbf4a35ea5021160773b1
```
查看容器状态
```
$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                      NAMES
c4196481196a        zabbix-java-gateway-4.0.1   "docker-entrypoint.sh"   16 seconds ago      Up 14 seconds       0.0.0.0:10052->10052/tcp   zabbix-java-gateway-4.0.1-01
```

#### zabbix-java-gateway 验证
```
# ps -ef|grep 10052
root     17455 14556  0 05:06 ?        00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 10052 -container-ip 172.17.0.3 -container-port 10052
# netstat -tunpl |grep 10052
tcp6       0      0 :::10052                :::*                    LISTEN      17455/docker-proxy
```
### zabbix-snmptraps 安装配置
#### 编写 Dockerfile 修改时区及编码并构建新镜像
Dockerfile
```
# cd /sdata/software/docker/data/zabbix-snmptraps
# vim Dockerfile
FROM zabbix/zabbix-snmptraps:centos-4.0.1
ENV LANG en_US.utf8
RUN yum install -y tzdata
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone \
```
构建镜像
```
# docker build -t zabbix-snmptraps-4.0.1 .
```
查看镜像
```
# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
zabbix-snmptraps-4.0.1          latest              1a04e096863a        48 seconds ago      427MB
```
#### 配置参数并启动 zabbix-snmptraps 容器
容器启动脚本
```
# cd /sdata/software/docker/data/zabbix-snmptraps
# mkdir snmptraps
# mkdir mibs
# ls
Dockerfile  mibs  snmptraps
# cd /sdata/software/docker/script/Docker-run
# vim zabbix-snmptraps-4.0.1-01.sh
#! /bin/bash

docker run --name zabbix-snmptraps-4.0.1-01 -t \
-v /sdata/software/docker/data/zabbix-snmptraps/snmptraps:/var/lib/zabbix/snmptraps:rw \
-v /sdata/software/docker/data/zabbix-snmptraps/mibs:/usr/share/snmp/mibs:ro \
-p 162:162/udp \
-d zabbix-snmptraps-4.0.1
```
启动容器
```
# sh zabbix-snmptraps-4.0.1-01.sh
```
查看容器状态
```
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                      NAMES
e34f0a76afb6        zabbix-snmptraps-4.0.1      "/usr/bin/supervisor…"   35 seconds ago      Up 33 seconds       0.0.0.0:162->162/udp       zabbix-snmptraps-4.0.1-01
```

#### zabbix-snmptraps 验证
```
# netstat -tunpl |grep 162
udp6       0      0 :::162                  :::*                                17941/docker-proxy  
```

### zabbix-server 安装配置
#### 创建 zabbix-server 持久化目录用于外部脚本调用
```
# cd /sdata/software/docker/data/zabbix-server
# mkdir ./alertscripts
# mkdir ./externalscripts
```

#### 编写 Dockerfile 修改时区及编码并构建新镜像
```
# vim Dockerfile
FROM zabbix/zabbix-server-mysql:centos-4.0.1
ENV LANG en_US.utf8
RUN yum install -y tzdata
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone \
# docker build -t zabbix-server-4.0.1 .
# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
zabbix-server-4.0.1             latest              698a1732d1a3        53 seconds ago      440MB
```
#### 配置参数并启动 zabbix-server 容器
```
# cd /sdata/software/docker/script/Docker-run
# vim zabbix-server-4.0.1-01.sh
#! /bin/bash

docker run --name zabbix-server-4.0.1-01 -t \
  -p 10051:10051 \
  --hostname zabbix-server \
  -e DB_SERVER_HOST="mysql-5.6-01"
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="passwd"\
  -e MYSQL_ROOT_PASSWORD="passwd" \
  -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
  --link mysql-5.6-01:mysql \
  --link zabbix-java-gateway-4.0.1-01:zabbix-java-gateway \
  --link zabbix-snmptraps-4.0.1-01:zabbix-snmptraps \
  --volumes-from zabbix-snmptraps-4.0.1-01 \
  -v /sdata/software/docker/data/zabbix-server/alertscripts:/usr/lib/zabbix/alertscripts \
  -v /sdata/software/docker/data/zabbix-server/externalscripts:/usr/lib/zabbix/externalscripts \
  -d zabbix-server-4.0.1
```
```
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                      NAMES
cb2acde54ddd        zabbix-server-4.0.1         "docker-entrypoint.sh"   7 seconds ago       Up 5 seconds        0.0.0.0:10051->10051/tcp   zabbix-server-4.0.1-01
```
#### zabbix-server 验证
```
# netstat -tunpl|grep 10051
tcp6       0      0 :::10051                :::*                    LISTEN      19156/docker-proxy  
```
```
# mysql -uzabbix -h10.0.0.138 -p
Enter password:
MySQL [(none)]> use zabbix
MySQL [zabbix]> show tables;
+----------------------------+
| Tables_in_zabbix           |
+----------------------------+
| acknowledges               |
| actions                    |
| alerts                     |
```

### zabbix-web 安装配置
#### 编写 Dockerfile 修改时区及编码并构建新镜像
```
# cd /sdata/software/docker/data/zabbix-web
# vim Dockerfile
FROM zabbix/zabbix-web-nginx-mysql:centos-4.0.1
ENV LANG en_US.utf8
RUN yum install -y tzdata
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo Asia/Shanghai > /etc/timezone \
```
```
# docker build -t zabbix-web-4.0.1 .
```
```
# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
zabbix-web-4.0.1                latest              4292c8bf8b13        About a minute ago   537MB
```
#### 配置参数并启动 zabbix-web 容器
```
# cd /sdata/software/docker/script/Docker-run
```
```
# vim zabbix-web-4.0.1-01.sh
#! /bin/bash

docker run --name zabbix-web-4.0.1-01 -t \
  -p 80:80 \
  --hostname zabbix-web \
  -e PHP_TZ="Asia/Shanghai" \
  -e DB_SERVER_HOST="mysql-5.6-01" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="passwd" \
  -e MYSQL_ROOT_PASSWORD="passwd" \
  --link mysql-5.6-01:mysql \
  --link zabbix-server-4.0.1-01:zabbix-server \
  -d zabbix-web-4.0.1
```
```
# sh zabbix-web-4.0.1-01.sh
3729e918c67dc3de6b7e6946089e5fc2d842dfc12e7ecd202c2d8dfd7d3e7201
```
```
[root@opd-zabbix-01 Docker-run]# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                         NAMES
3729e918c67d        zabbix-web-4.0.1            "docker-entrypoint.sh"   5 seconds ago       Up 3 seconds        0.0.0.0:80->80/tcp, 443/tcp   zabbix-web-4.0.1-01
cb2acde54ddd        zabbix-server-4.0.1         "docker-entrypoint.sh"   25 minutes ago      Up 25 minutes       0.0.0.0:10051->10051/tcp      zabbix-server-4.0.1-01
e34f0a76afb6        zabbix-snmptraps-4.0.1      "/usr/bin/supervisor…"   17 hours ago        Up 17 hours         0.0.0.0:162->162/udp          zabbix-snmptraps-4.0.1-01
c4196481196a        zabbix-java-gateway-4.0.1   "docker-entrypoint.sh"   18 hours ago        Up 18 hours         0.0.0.0:10052->10052/tcp      zabbix-java-gateway-4.0.1-01
89baa5ab3918        mysql-centos-5.6            "docker-entrypoint.s…"   20 hours ago        Up 20 hours         0.0.0.0:3306->3306/tcp        mysql-5.6-01
```
### zabbix-web 验证
```
# ps -ef|grep zabbix
root     17483 17464  0 05:06 pts/0    00:00:00 su zabbix -s /bin/bash -c /usr/sbin/zabbix_java_gateway
polkitd  17521 17483  0 05:06 ?        00:01:17 /usr/bin/java -server -Dlogback.configurationFile=/etc/zabbix/zabbix_java_gateway_logback.xml -classpath /usr/sbin/zabbix_java/lib:/usr/sbin/zabbix_java/lib/slf4j-api-1.6.1.jar:/usr/sbin/zabbix_java/lib/android-json-4.3_r3.1.jar:/usr/sbin/zabbix_java/lib/logback-core-0.9.27.jar:/usr/sbin/zabbix_java/lib/logback-classic-0.9.27.jar:/usr/sbin/zabbix_java/bin/zabbix-java-gateway-4.0.1.jar com.zabbix.gateway.JavaGateway
root     19183 19164  0 22:23 pts/0    00:00:00 su zabbix -s /bin/bash -c /usr/sbin/zabbix_server --foreground -c /etc/zabbix/zabbix_server.conf
polkitd  19368 19183  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server --foreground -c /etc/zabbix/zabbix_server.conf
polkitd  19370 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: configuration syncer [synced configuration in 0.017979 sec, idle 60 sec]
polkitd  19371 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: alerter #1 started
polkitd  19372 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: alerter #2 started
polkitd  19373 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: alerter #3 started
polkitd  19374 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: housekeeper [startup idle for 30 minutes]
polkitd  19375 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: timer #1 [updated 0 hosts, suppressed 0 events in 0.001004 sec, idle 59 sec]
polkitd  19376 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: http poller #1 [got 0 values in 0.000915 sec, idle 5 sec]
polkitd  19377 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: discoverer #1 [processed 0 rules in 0.000974 sec, idle 60 sec]
polkitd  19378 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: history syncer #1 [processed 0 values, 0 triggers in 0.000029 sec, idle 1 sec]
polkitd  19379 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: history syncer #2 [processed 0 values, 0 triggers in 0.000029 sec, idle 1 sec]
polkitd  19381 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: history syncer #3 [processed 0 values, 0 triggers in 0.000030 sec, idle 1 sec]
polkitd  19382 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: history syncer #4 [processed 0 values, 0 triggers in 0.000029 sec, idle 1 sec]
polkitd  19384 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: escalator #1 [processed 0 escalations in 0.001589 sec, idle 3 sec]
polkitd  19386 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: proxy poller #1 [exchanged data with 0 proxies in 0.000029 sec, idle 5 sec]
polkitd  19388 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: self-monitoring [processed data in 0.000036 sec, idle 1 sec]
polkitd  19389 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: task manager [processed 0 task(s) in 0.000578 sec, idle 4 sec]
polkitd  19390 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: poller #1 [got 0 values in 0.000016 sec, idle 5 sec]
polkitd  19391 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: poller #2 [got 0 values in 0.000020 sec, idle 5 sec]
polkitd  19393 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: poller #3 [got 0 values in 0.000019 sec, idle 5 sec]
polkitd  19397 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: poller #4 [got 0 values in 0.000031 sec, idle 5 sec]
polkitd  19398 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: poller #5 [got 0 values in 0.000029 sec, idle 5 sec]
polkitd  19399 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: unreachable poller #1 [got 0 values in 0.000087 sec, idle 1 sec]
polkitd  19400 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: trapper #1 [processed data in 0.000000 sec, waiting for connection]
polkitd  19404 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: trapper #2 [processed data in 0.000000 sec, waiting for connection]
polkitd  19405 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: trapper #3 [processed data in 0.000000 sec, waiting for connection]
polkitd  19406 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: trapper #4 [processed data in 0.000000 sec, waiting for connection]
polkitd  19408 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: trapper #5 [processed data in 0.000000 sec, waiting for connection]
polkitd  19412 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: icmp pinger #1 [got 0 values in 0.000026 sec, idle 5 sec]
polkitd  19415 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: alert manager #1 [sent 0, failed 0 alerts, idle 5.009056 sec during 5.009131 sec]
polkitd  19418 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: preprocessing manager #1 [queued 0, processed 0 values, idle 5.005807 sec during 5.005903 sec]
polkitd  19419 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: preprocessing worker #1 started
polkitd  19421 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: preprocessing worker #2 started
polkitd  19422 19368  0 22:26 ?        00:00:00 /usr/sbin/zabbix_server: preprocessing worker #3 started
# netstat -tunpl|grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      19783/docker-proxy
```
浏览器访问 10.0.0.138



## 客户端 zabbix-agentd 安装
由于使用 docker 版本的 zabbix-agent 容易引起容器内外监控指标混乱的问题，所以此处使用原有编译安装的方法安装zabbix-agent。

## zabbix-server 主机上的 zabbix-agent 安装
### 被动模式手动安装
解压编译
```
tar -xvz -f  /sdata/software/zabbix-4.0.1.tar.gz -C /sdata/software
mkdir /sdata/software/zabbix
cd /sdata/software/zabbix-4.0.1
./configure --prefix=/sdata/software/zabbix --enable-agent
```
添加 zabbix 用户并创建日志文件
```
useradd zabbix
mkdir /sdata/var/log
vim zabbix_agentd.log
chown zabbix:zabbix /sdata/var/log/zabbix_agentd.log
```
配置 zabbix-agentd，注意：Server要填 zabbix-server 在 docker 容器中的 ip
```
cd /sdata/software/zabbix/etc
vim /sdata/software/zabbix/etc/zabbix_agentd.conf
LogFile=/sdata/var/log/zabbix_agentd.log
Server=172.17.0.5
Timeout=30
Include=/sdata/software/zabbix/etc/zabbix_agentd.conf.d/*.conf
```
启动 zabbix-agent
```
cd /sdata/software/zabbix/sbin
./zabbix_agentd
```
zabbix 后台修改 zabbix-server 的 agent 配置，agent interfaces 的 IP 要改为 zabbix-server 的实际 IP，而不是 docker 容器的 IP。
