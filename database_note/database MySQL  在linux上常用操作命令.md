# database MySQL 在linux上常用操作命令
## MySQL 安装配置
[MySQL 在 linux 上的配置安装](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEMySQL%EF%BC%88MariaDB%EF%BC%89.MD)
## MySQL 在 linux 上一些常用操作命令
注意：linux上命令行必须以;结束，不然回车会自动换行
### 查看引擎
```
MariaDB [(none)]> show engines;
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                                    | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| CSV                | YES     | CSV storage engine                                                         | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                                      | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables                  | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears)             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                                      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                                     | NO           | NO   | NO         |
| FEDERATED          | YES     | FederatedX pluggable storage engine                                        | YES          | NO   | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                                         | NO           | NO   | NO         |
| Aria               | YES     | Crash-safe tables with MyISAM heritage                                     | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
10 rows in set (0.00 sec)

```
### 转换表的引擎(注意表的结构，如外键)
#### 方法1：直接改
```
MariaDB [personalFinancialDB]> ALTER TABLE persons engine=MyISAM;
ERROR 1217 (23000): Cannot delete or update a parent row: a foreign key constraint fails
```
会耗费大量资源，最好手动进行表复制
#### 方法2：导出导入
可用mysqldump导出文件，然后CREATE TABLE来给新表设置引擎，注意改表名
```
# 表名修改
RENAME TABLE table_a TO table_b
```
#### 方法3：创建查询
不需要导出整个表的文件，可在存储引擎中直接创建一个新表来进行原表的全量复制，表很大时可以只选取部分数据。

### 查看用户权限
查看某位用户的权限
show grants for [username];
```
MariaDB [(none)]> show grants for sharonli;
+------------------------------------------------------------------------------------------------------------------+
| Grants for sharonli@%                                                                                            |
+------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'sharonli'@'%' IDENTIFIED BY PASSWORD '*9F4094FBC28842418DE5A9DF5413521CD8E4BDE1' |
+------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```
### 查看正在执行的查询语句
```
MariaDB [(none)]> show processlist;
+----+----------+-----------+------+---------+------+-------+------------------+----------+
| Id | User     | Host      | db   | Command | Time | State | Info             | Progress |
+----+----------+-----------+------+---------+------+-------+------------------+----------+
|  2 | sharonli | localhost | NULL | Sleep   | 1223 |       | NULL             |    0.000 |
| 15 | sharonli | localhost | NULL | Query   |    0 | NULL  | show processlist |    0.000 |
+----+----------+-----------+------+---------+------+-------+------------------+----------+
2 rows in set (0.07 sec)

```
### 查看慢查询次数
默认为超过10s的慢查询
```
MariaDB [(none)]> show status like 'slow_queries';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 0     |
+---------------+-------+
1 row in set (0.20 sec)
```
查询默认慢查询时间
```
MariaDB [(none)]> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.19 sec)
```
修改默认慢查询时间
```
MariaDB [(none)]> set long_query_time= 1;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
1 row in set (0.00 sec)
```
### 查看表的相关信息
```
MariaDB [personalFinancialDB]> show table status like 'persons'\G；
*************************** 1. row ***************************
           Name: persons
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 3
 Avg_row_length: 5461
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 9437184
 Auto_increment: 4
    Create_time: 2018-08-11 18:08:15
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_unicode_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.01 sec)

    -> 

```
查看表的行数
```
MariaDB [personalFinancialDB]> select count(*) from persons;
+----------+
| count(*) |
+----------+
|        3 |
+----------+
1 row in set (0.23 sec)

```

## mysql 性能测试相关
### 输出指标
```
show global status
```
过程：装载数据、系统预热、执行测试、记录结果</br>
重要指标：CPU使用率、磁盘I/O、网络流量统计、show global status的一些输出</br>
应保存：每轮测试的测试结果、配置文件、测试指标、测试用脚本、相关说明</br>
### 性能差时可查
```
show engine innodb status;
show full processlist;
```
### 测试工具
集成：ab http_load jmeter</br>
单组件：sysbench 多线程压测(在mysql用户工具包中)</br>
mysql 内置函数：benchmark（） 测试某些特定操作的执行速度（要清楚原理）</br>
```
# sysbench cpu基准测试
$ cat /proc/cpuinfo
processor	: 0
vendor_id	: AuthenticAMD
cpu family	: 16
model		: 6
model name	: AMD Turion(tm) II Dual-Core Mobile M520
stepping	: 2
microcode	: 0x1000098
cpu MHz		: 2294.279
cache size	: 512 KB
physical id	: 0
siblings	: 1
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fdiv_bug	: no
f00f_bug	: no
coma_bug	: no
fpu		: yes
fpu_exception	: yes
cpuid level	: 5
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm 3dnowext 3dnow constant_tsc art tsc_reliable nonstop_tsc pni cx16 x2apic popcnt hypervisor lahf_lm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw retpoline_amd ibp_disable vmmcall
bogomips	: 4588.55
clflush size	: 64
cache_alignment	: 64
address sizes	: 40 bits physical, 48 bits virtual
power management:

$ sysbench --test=cpu --cpu-max=prime=20000 run

```
## MySQL服务器状态
### mysqladmin 管理命令
mysqladmin 工具的使用格式
```
mysqladmin [option] command [command option] command ......
option 选项：
     -c  number 自动运行次数统计，必须和 -i 一起使用
     -i   number 间隔多长时间重复执行
```
每个两秒查看一次服务器的状态，总共重复5次
```
./mysqladmin -uroot -p  -i 2 -c 5 status
```

## 载入SQL文件
```
# 登录mysql后
source sql文件绝对路径
```
