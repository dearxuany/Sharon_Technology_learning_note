# zabbix 使用 Percona Monitoring Plugins 监控 MySQL
zabbix mysql相关监控:</br>
https://www.zabbix.com/cn/integrations/mysql </br>
Percona Monitoring Plugins for Zabbix: </br>
https://www.percona.com/doc/percona-monitoring-plugins/1.1/zabbix/index.html </br>
最新版本 Percona Monitoring Plugins 下载 </br>
https://www.percona.com/downloads/percona-monitoring-plugins/LATEST/ </br>
## 系统版本
```
Linux version 3.13.0-32-generic (buildd@kissel) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) )
zabbix_agentd (daemon) (Zabbix) 4.0.1
PHP 5.5.9-1ubuntu4.27 (cli)
php5-mysql
mysql  Ver 14.14 Distrib 5.6.33, for debian-linux-gnu (x86_64) using  EditLine wrapper
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
由于文件会被放置在/var/lib/zabbix/percona/scrpts 和 /var/lib/zabbix/percona/templates 下，所以最好先新建这两个目录。</br>
/var/lib/zabbix/percona/scripts/这里面的两个文件，sh的脚本是监控获取MySQL状态的，php文件是配置连接数据库用户名密码的。用shell来调用PHP。</br>
/var/lib/zabbix/percona/templates/这里面的两个文件，conf文件是要放在agent端/etc/zabbix/zabbix_agentd.d/下面的，XML文件是模版文件。</br>
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
查看 zabbix的配置文件，看是否开启下级配置文件，没有则添加
```
vim zabbix_agentd.conf

Include=/sdata/software/zabbix/etc/zabbix_agentd.conf.d/*.conf
```
复制 userparameter_percona_mysql.conf 到 zabbix agent的配置目录内
```
/sdata/software/zabbix/etc/zabbix_agentd.conf.d# cp /var/lib/zabbix/percona/templates/userparameter_percona_mysql.conf ./
```
重启 zabbix agent
```
# service zabbix-agentd restart
Stopping Zabbix agent daemon: zabbix_agentd
Starting Zabbix agent daemon: zabbix_agentd
```
## 配置 MySQL connectivity 代理
新建 /var/lib/zabbix/percona/scripts/ss_get_mysql_stats.php.cnf 文件，填入以下内容
```

```


