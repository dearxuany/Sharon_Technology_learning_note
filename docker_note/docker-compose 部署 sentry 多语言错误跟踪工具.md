# docker-compose 部署 sentry 多语言错误跟踪工具
## 依赖
docker docker-ce-18.09.6
docker-compose version 1.25.4
git

## sentry 安装
https://sentry.io/welcome/
### 代码下载
从 git 下下载源代码
```
# mkdir -p /sdata/app
# cd /sdata/app
# git clone https://github.com/getsentry/onpremise.git
```
由于现阶段 sentry 10 还在开发阶段会导致 build 失败，故需切换到 9.1.2 稳定版本
```
cd /sdata/app/onpremise
git pull
git checkout tags/9.1.2
git checkout master
git pull
```
目录结构
```
cd /sdata/app/onpremise
[root@opd-sentry-01 onpremise]# tree -L 3
.
├── cron
│   ├── Dockerfile
│   └── entrypoint.sh 
├── docker-compose.yml
├── install.sh  # 部署脚本
├── LICENSE
├── README.md
├── sentry
│   ├── config.example.yml  #  sentry 环境配置文件
│   ├── Dockerfile
│   ├── requirements.example.txt  # python 依赖包
│   └── sentry.conf.example.py    # django 配置文件
└── test.sh


2 directories, 11 files
```

### 配置修改
#### 邮件发送配置
修改 /sdata/app/onpremise/sentry/config.yml
```
mail.host: 'smtp.exmail.qq.com'
mail.port: 587  # 使用 587 tsl 必须为 true，注腾信企业邮箱 465 为 ssl 而 sentry 仅支持 tsl 
mail.username: 'sentry@domainname.com'
mail.password: 'passwd'
mail.use-tls: true   # 使用 tsl 必须为 true
mail.from: 'sentry@domainname.com' # 必须和 username 一致
```
修改 django 原生配置延长 socket 超时时间，添加以下代码
```
# vim /sdata/app/onpremise/sentry/sentry.conf.py

import socket

socket.setdefaulttimeout(20)
```
#### 添加钉钉提醒插件
注：由于钉钉版本更新，新增安全设置要求提供关键字参数，故此版本 dingtalk plugins 暂时无法使用
```
# vim /sdata/app/onpremise/sentry/requirements.txt
# Add plugins here
sentry-dingding~=0.0.3
django-smtp-ssl~=1.0
redis-py-cluster==1.3.4
```

## 构建 sentry
使用 sentry 官方脚本构建容器</br>
注意：容器构建速度取决于镜像拉取速度，中途断开可重新执行，脚本内部已有判断，会在失败处继续向后构建
```
# cd /sdata/app/onpremise
# ./install.sh
```
构建成功后会要求输入管理邮箱和密码
```
Would you like to create a user account now? [Y/n]: y
Email: sentry@ingbaobei.com
Password:
Repeat for confirmation:
User created: sentry@domainname.com
Added to organization: sentry
```
查看容器
```
[root@opd-sentry-01 ~]# docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS                          NAMES
ba422944b651        sentry-onpremise-local                 "/bin/sh -c 'exec /d…"   10 minutes ago      Up 10 minutes       0.0.0.0:9000->9000/tcp         sentry_onpremise_web_1
2f921548a416        sentry-onpremise-local                 "/bin/sh -c 'exec /d…"   10 minutes ago      Up 10 minutes       9000/tcp                       sentry_onpremise_cron_1
3527d97ed094        sentry-cleanup-onpremise-local         "/entrypoint.sh '0 0…"   10 minutes ago      Up 10 minutes       9000/tcp                       sentry_onpremise_sentry-cleanup_1
1435e98b78cb        sentry-onpremise-local                 "/bin/sh -c 'exec /d…"   10 minutes ago      Up 10 minutes       9000/tcp                       sentry_onpremise_post-process-forwarder_1
d79ea01c4520        sentry-onpremise-local                 "/bin/sh -c 'exec /d…"   10 minutes ago      Up 10 minutes       9000/tcp                       sentry_onpremise_worker_1
1e0c8eafda51        snuba-cleanup-onpremise-local          "/entrypoint.sh '*/5…"   10 minutes ago      Up 10 minutes       1218/tcp                       sentry_onpremise_snuba-cleanup_1
ca89b580c30b        symbolicator-cleanup-onpremise-local   "/entrypoint.sh '55 …"   11 minutes ago      Up 10 minutes       3021/tcp                       sentry_onpremise_symbolicator-cleanup_1
6b41986b9eaa        getsentry/snuba:latest                 "./docker_entrypoint…"   11 minutes ago      Up 10 minutes       1218/tcp                       sentry_onpremise_snuba-replacer_1
8cd65c6cbd23        getsentry/snuba:latest                 "./docker_entrypoint…"   11 minutes ago      Up 10 minutes       1218/tcp                       sentry_onpremise_snuba-api_1
278faa5fb06e        getsentry/snuba:latest                 "./docker_entrypoint…"   11 minutes ago      Up 10 minutes       1218/tcp                       sentry_onpremise_snuba-consumer_1
2fd091fcdca6        memcached:1.5-alpine                   "docker-entrypoint.s…"   11 minutes ago      Up 10 minutes       11211/tcp                      sentry_onpremise_memcached_1
3ed4736dfd57        postgres:9.6                           "docker-entrypoint.s…"   11 minutes ago      Up 10 minutes       5432/tcp                       sentry_onpremise_postgres_1
a077c40d5245        tianon/exim4                           "docker-entrypoint.s…"   11 minutes ago      Up 10 minutes       25/tcp                         sentry_onpremise_smtp_1
4aa3b002b152        getsentry/symbolicator:latest          "/bin/bash /docker-e…"   11 minutes ago      Up 10 minutes       3021/tcp                       sentry_onpremise_symbolicator_1
ad78d1391c9f        confluentinc/cp-kafka:5.1.2            "/etc/confluent/dock…"   12 minutes ago      Up 10 minutes       9092/tcp                       sentry_onpremise_kafka_1
4158b2087227        yandex/clickhouse-server:19.11         "/entrypoint.sh"         12 minutes ago      Up 10 minutes       8123/tcp, 9000/tcp, 9009/tcp   sentry_onpremise_clickhouse_1
0064489a8e0a        redis:5.0-alpine                       "docker-entrypoint.s…"   12 minutes ago      Up 10 minutes       6379/tcp                       sentry_onpremise_redis_1
63db8c775b88        confluentinc/cp-zookeeper:5.1.2        "/etc/confluent/dock…"   12 minutes ago      Up 10 minutes       2181/tcp, 2888/tcp, 3888/tcp   sentry_onpremise_zookeeper_1
```

启动容器
```
[root@opd-sentry-01 onpremise]# docker-compose up -d
Starting sentry_onpremise_zookeeper_1            ... done
Starting sentry_onpremise_memcached_1            ... done
Starting sentry_onpremise_clickhouse_1           ... done
Starting sentry_onpremise_redis_1                ... done
Starting sentry_onpremise_smtp_1                 ... done
Starting sentry_onpremise_symbolicator_1         ... done
Starting sentry_onpremise_postgres_1             ... done
Creating sentry_onpremise_symbolicator-cleanup_1 ... done
Starting sentry_onpremise_kafka_1                ... done
Starting sentry_onpremise_snuba-replacer_1       ... done
Starting sentry_onpremise_snuba-api_1            ... done
Starting sentry_onpremise_snuba-consumer_1       ... done
Creating sentry_onpremise_snuba-cleanup_1        ... done
Creating sentry_onpremise_worker_1                 ... done
Creating sentry_onpremise_post-process-forwarder_1 ... done
Creating sentry_onpremise_sentry-cleanup_1         ... done
Creating sentry_onpremise_web_1                    ... done
Creating sentry_onpremise_cron_1                   ... done
```
访问 sentry
```
http://opd-sentry-01.snail:9000/auth/login/sentry
```
## nginx 代理
```
# vim sentry.domainname.com.conf
server {
  listen       80;
  server_name  sentry.domainname.com;


  if ($host = "sentry.domainname.com"){
    return 301 https://sentry.domainname.com$request_uri;
  }
}


server {
  listen       443 ssl;
  server_name  sentry.domainname.com;
  access_log /sdata/var/log/nginx/domainname.com/sentry.domainname.com_access.log main;
  error_log /sdata/var/log/nginx/domainname.com/sentry.domainname.com_error.log error;


  ssl_protocols         SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2;
  ssl_certificate       conf.d/keys/domainname.com.pem;
  ssl_certificate_key   conf.d/keys/domainname.com.key;
  ssl_session_timeout 5m;
  ssl_prefer_server_ciphers on;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;


  location / {
      proxy_pass http://opd-sentry-01.snail:9000;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $remote_addr;
      proxy_connect_timeout 600;
      proxy_read_timeout 600;
      proxy_send_timeout 600;
      #client_max_body_size 1024M; # Set higher depending on your needs
  }
}
```
