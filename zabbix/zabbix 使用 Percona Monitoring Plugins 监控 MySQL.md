# zabbix 使用 Percona Monitoring Plugins 监控 MySQL
zabbix mysql相关监控:</br>
https://www.zabbix.com/cn/integrations/mysql </br>
Percona Monitoring Plugins for Zabbix:</br>
https://www.percona.com/doc/percona-monitoring-plugins/1.1/zabbix/index.html </br>
最新版本 Percona Monitoring Plugins 下载
https://www.percona.com/downloads/percona-monitoring-plugins/LATEST/ </br>
## 系统版本
```
Linux version 3.13.0-32-generic (buildd@kissel) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) )
zabbix_agentd (daemon) (Zabbix) 4.0.1
PHP 5.5.9-1ubuntu4.27 (cli)
php5-mysql
```
## 配置 zabbix agent
源码安装，下载 percona-zabbix-templates-1.1.5-1.noarch.rpm
```
wget https://www.percona.com/downloads/percona-monitoring-plugins/1.1.5/percona-zabbix-templates-1.1.5-1.noarch.rpm
```
由于ubuntu无法使用rpm包安装，要用 alien 将 rpm 转为 deb
```
alien percona-zabbix-templates-1.1.5-1.noarch.rpm
```
由于文件会被放置在/var/lib/zabbix/percona/scrpts 和 /var/lib/zabbix/percona/templates 下，所以最好先新建这两个目录
```
/var/lib/zabbix/percona# tree -L 2
.
├── scripts
│   ├── get_mysql_stats_wrapper.sh
│   └── ss_get_mysql_stats.php
└── templates
    ├── userparameter_percona_mysql.conf
    └── zabbix_agent_template_percona_mysql_server_ht_2.0.9-sver1.1.5.xml

2 directories, 4 files
```
