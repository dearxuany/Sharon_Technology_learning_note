# canal 通过 mysql binlog 增量同步 mysql 数据到 elasticsearch
## canal 依赖
### java 1.8+
```
# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```
### mysql 配置
由于使用 aliyun rds，阿里云底层不允许使用只读库获取 binlog，故 canal-server 必须链接 rds mysql 主库。
#### binlog 输出格式设置为 row
确认 binlog 开启
```
show variables like '%log_bin%';
```
确认 binlog 输出模式为 row
```
show variables like 'binlog_format'
```
#### 授权 canal 链接 mysql 用户拥有  MySQL slave 权限
考虑安全问题，注意修改 % 只授权给限定 ip 使用该账户
```
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```
查看指定用户权限
```
SHOW GRANTS FOR 'canal'@'%';
```

### 阿里云 ram 账号配置
#### 问题描述
aliyun RDS有自己的binlog日志清理策略，这个策略相比于用户自建mysql会更加激进，默认应该是18小时就会清理binlog并上传到oss上，如果canal任务停止超过18小时会导致 binlog 获取位点不对致报错。
#### 解决办法
RDS默认提供了一段时间oss binlog的下载能力，canal可以识别位点中的时间戳，对比一下RDS中show binary logs里最早的一条binlog，如果不满足则会通过oss接口进行下载到本机进行解析，追平历史binlog之后再切换到RDS binlog中继续消费。
#### Ram 账号调用 DescribeBinlogFiles api 权限配置
canal 中的代码通过 RAM 账号调用 aliyun rds 的 DescribeBinlogFiles 接口来获取 oss 中的备份 binlog，故需要在 canal 配置文件中添加 ram 账号的 accesskey 和 secret。基于安全角度考虑，aliyun ram 只读子账号无法下载备份文件，可以通过RAM控制台给只读子账号添加下载备份文件的权限。
```
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "rds:Describe*",
                "rds:ModifyBackupPolicy"
            ],
            "Resource": [
                "acs:rds:*:*:*/rdsInstanceID"
            ]
        }
    ],
    "Version": "1"
}
```
canal  aliyun RDS QuickStart https://github.com/alibaba/canal/wiki/aliyun-RDS-QuickStart </br>
aliyun RDS api DescribeBinlogFiles https://help.aliyun.com/document_detail/26291.html </br>
rds 下载数据备份和日志备份 https://help.aliyun.com/document_detail/98819.html?spm=a2c4g.11174283.6.721.3b3a5b83zuYUeT </br>
rds 添加下载备份文件权限给只读子账号  https://help.aliyun.com/document_detail/100043.html?spm=a2c4g.11186623.2.11.4abc7d96sfgbQ0#concept-qmt-zxm-cgb </br>
rds RAM 资源授权  https://help.aliyun.com/document_detail/26307.html?spm=a2c4g.11186631.6.1582.57d05c85zA7E5d </br>

## canal-server 部署
canal 官网  https://github.com/alibaba/canal/wiki/QuickStart
### 安装包
此处使用 https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.deployer-1.1.4.tar.gz 版本
```
# canal-server 目录结构

# tree -L 2
.
├── bin
│   ├── canal.pid
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── canal_local.properties
│   ├── canal.properties  
│   ├── example
│   ├── logback.xml
│   ├── metrics
│   └── spring
├── lib
│   ├── aopalliance-1.0.jar
│   ├── aviator-2.2.1.jar
│   ├── ...
│   └── zookeeper-3.4.5.jar
└── logs
    ├── canal
    └── example


# 配置文件目录

# tree -L 3
.
├── canal_local.properties  # 使用 canal.admin 时会用到，主要替代 canal.properties 接管 canal-server 管理
├── canal.properties  # canal-server 主配置文件，主要配置 canal-server 服务本身的一些属性，如监听 ip、端口等
├── example
│   ├── h2.mv.db
│   ├── h2.trace.db
│   ├── instance.properties  # 定义 canal-server 实例化后的配置，例如要获取的库表信息等
│   └── meta.dat  # 为一个标准json，记录了canal 正在获取的 binlog 文件名及起始位置，canal-server 重启后会根据此文件找 binlog
├── logback.xml
├── metrics
│   └── Canal_instances_tmpl.json
└── spring
    ├── base-instance.xml
    ├── default-instance.xml
    ├── file-instance.xml
    ├── group-instance.xml
    ├── memory-instance.xml
    └── tsdb
        ├── h2-tsdb.xml
        ├── mysql-tsdb.xml
        ├── sql
        └── sql-map
```
使用 ansible playbook 部署
```
---

- hosts: prd_canal-01
  become: yes
  become_user: root

  tasks:
  - name: Create Goal Dir
    file: path=/sdata/usr/local/src state=directory mode=0755

  - name: Download package
    get_url:
      url: https://pkg.domainname.com/canal/canal.deployer-1.1.4.tar.gz
      dest: /sdata/usr/local/src/
      mode: '0644'
      url_username: pkg
      url_password: passwd

  - name: create canal dir and untar canal
    shell: mkdir /sdata/usr/local/canal-server && tar -xvzf /sdata/usr/local/src/canal.deployer-1.1.4.tar.gz -C /sdata/usr/local/canal-server

  - name: add canal user
    shell: chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow && useradd canal && chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow

  - name: change canal owner
    file: path=/sdata/usr/local/canal-server owner=canal group=canal recurse=yes

  - name: creat canal log dir
    file: path=/sdata/var/log/canal-server state=directory owner=canal group=canal mode=0755  
```
### canal-server 主配置修改
修改 canal-server 主机及绑定端口
```
$ canal.properties

# tcp bind ip
canal.ip =
# register ip to zookeeper
canal.register.ip =
canal.port = 11111
canal.metrics.pull.port = 11112
```

### canal-server 实例配置修改
```
$ vim ./conf/example/instance.properties
```
设置 mysql 链接实例及端口
```
# position info
canal.instance.master.address=hostname:3306
canal.instance.master.journal.name=  # 从指定的binlog文件开始读取数据
canal.instance.master.position= # 指定偏移量,binlog和偏移量也可以不指定，则canal-server会从当前的位置开始读取
canal.instance.master.timestamp= # mysql主库链接时起始的binlog的时间戳，默认:无，canal 会根据 timestamp 在 oss 上找 rds 的备份 binlog 和位点
canal.instance.master.gtid=
```
设置下载 OSS 备份 binlog 的 ram 账号授权信息
```
# rds oss binlog
canal.instance.rds.accesskey=accesskey
canal.instance.rds.secretkey=secretkey
canal.instance.rds.instanceId=instanceId
```
设置 canal 链接 mysql 的用户密码
```
# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=passwd
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
```
设置过滤要获取的指定表，获取多个不同表用英文逗号隔开；aliyun rds 的链接需注意设置 mysql\..* 为黑名单，不获取该库 binlog 否则会报错 
```
# table regex
# canal.instance.filter.regex=.*\\..*
canal.instance.filter.regex=enterprise.t_enterprise_chat_content,enterprise.t_enterprise_external_user_info,enterprise.t_enterprise_user_info
# table black regex
canal.instance.filter.black.regex=mysql\..*
```
不是用 mq 注释 mq 相关配置
```
# mq config
# canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
# canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
```
### 修改日志相关保存路径
```
$ vim logback.xml

<configuration scan="true" scanPeriod=" 5 seconds">
        <jmxConfigurator />
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
                <encoder>
                        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{56} - %msg%n
                        </pattern>
                </encoder>
        </appender>


        <appender name="CANAL-ROOT" class="ch.qos.logback.classic.sift.SiftingAppender">
                <discriminator>
                        <Key>destination</Key>
                        <DefaultValue>canal</DefaultValue>
                </discriminator>
                <sift>
                        <appender name="FILE-${destination}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                                <File>/sdata/var/log/canal-server/${destination}/${destination}.log</File>
                                <rollingPolicy
                                                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                                        <!-- rollover daily -->
                                        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                                                <!-- or whenever the file size reaches 100MB -->
                                                <maxFileSize>512MB</maxFileSize>
                                        </timeBasedFileNamingAndTriggeringPolicy>
                                        <maxHistory>60</maxHistory>
                                </rollingPolicy>
                                <encoder>
                                        <pattern>
                                                %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{56} - %msg%n
                                        </pattern>
                                </encoder>
                        </appender>
                </sift>
        </appender>


        <appender name="CANAL-META" class="ch.qos.logback.classic.sift.SiftingAppender">
                <discriminator>
                        <Key>destination</Key>
                        <DefaultValue>canal</DefaultValue>
                </discriminator>
                <sift>
                        <appender name="META-FILE-${destination}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                                <File>/sdata/var/log/canal-server/${destination}/meta.log</File>
                                <rollingPolicy
                                                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                                        <!-- rollover daily -->
                                        <fileNamePattern>../logs/${destination}/%d{yyyy-MM-dd}/meta-%d{yyyy-MM-dd}-%i.log.gz</fileNamePattern>
                                        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                                                <!-- or whenever the file size reaches 100MB -->
                                                <maxFileSize>32MB</maxFileSize>
                                        </timeBasedFileNamingAndTriggeringPolicy>
                                        <maxHistory>60</maxHistory>
                                </rollingPolicy>
                                <encoder>
                                        <pattern>
                                                %d{yyyy-MM-dd HH:mm:ss.SSS} - %msg%n
                                        </pattern>
                                </encoder>
                        </appender>
                </sift>
        </appender>


        <logger name="com.alibaba.otter.canal.instance" additivity="false">
                <level value="INFO" />
                <appender-ref ref="CANAL-ROOT" />
        </logger>
        <logger name="com.alibaba.otter.canal.deployer" additivity="false">
                <level value="INFO" />
                <appender-ref ref="CANAL-ROOT" />
        </logger>
        <logger name="com.alibaba.otter.canal.meta.FileMixedMetaManager" additivity="false">
                <level value="INFO" />
                <appender-ref ref="CANAL-META" />
        </logger>
        <logger name="com.alibaba.otter.canal.kafka" additivity="false">
                <level value="INFO" />
                <appender-ref ref="CANAL-ROOT" />
        </logger>


        <root level="WARN">
                <!-- <appender-ref ref="STDOUT"/>  -->
                <appender-ref ref="CANAL-ROOT" />
        </root>
</configuration>
```



## 启动验证
```
[canal@alihn1-prd-canal-01 canal]$ tail -f canal.log
2020-07-01 16:19:49.526 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2020-07-01 16:19:49.562 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2020-07-01 16:19:49.573 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2020-07-01 16:19:49.607 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[172.**.20.229(172.**.20.229):11111]
2020-07-01 16:19:50.640 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......
^C

[canal@alihn1-prd-canal-01 example]$ tail -f example.log
2020-07-01 17:31:51.442 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
2020-07-01 17:31:51.534 [destination = example , address = mysqlhostname/172.**.244.**:3307 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2020-07-01 17:31:51.535 [destination = example , address = mysqlhostname/172.**.244.**:3307 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
2020-07-01 17:31:54.441 [destination = example , address = mysqlhostname/172.**.244.**:3307 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=mysql-bin.001967,position=321450185,serverId=3251810213,gtid=,timestamp=1593595911000] cost : 2900ms , the next step is binlog dump
2020-07-01 17:39:47.396 [New I/O server worker #1-1] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - subscribe filter change to enterprise.t_enterprise_chat_content
2020-07-01 17:39:47.397 [New I/O server worker #1-1] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^enterprise.t_enterprise_chat_content$
2020-07-01 17:43:56.190 [New I/O server worker #1-2] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - subscribe filter change to enterprise.t_enterprise_chat_content
2020-07-01 17:43:56.190 [New I/O server worker #1-2] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^enterprise.t_enterprise_chat_content$
2020-07-01 17:45:41.886 [New I/O server worker #1-3] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - subscribe filter change to enterprise.t_enterprise_chat_content
2020-07-01 17:45:41.886 [New I/O server worker #1-3] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^enterprise.t_enterprise_chat_content$



[canal@alihn1-prd-canal-01 example]$ tail -f meta.log
2020-07-01 17:39:47.432 - clientId:1001 cursor:[mysql-bin.001967,321450468,1593595911000,3251810213,] address[mysqlhostname/172.**.244.**:3307]
2020-07-01 17:39:48.432 - clientId:1001 cursor:[mysql-bin.001967,321508317,1593595912000,3251810213,] address[mysqlhostname/172.**.244.**:3307]
2020-07-01 17:43:56.432 - clientId:1001 cursor:[mysql-bin.001967,321577769,1593595913000,3251810213,] address[mysqlhostname/172.**.244.**:3307]
2020-07-01 17:45:42.432 - clientId:1001 cursor:[mysql-bin.001967,321638391,1593595914000,3251810213,] address[mysqlhostname/172.**.244.**:3307]
