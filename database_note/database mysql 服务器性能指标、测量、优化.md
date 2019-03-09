# mysql 服务器性能指标、测量、优化
分析 mysql 性能时，一般指分析 mysql 的查询
## 性能简介
### 定义
服务器性能为“响应时间”，“响应时间”分为“执行时间”和“等待时间”。</br>
* 执行时间：什么任务执行时间最长
* 等待时间：任务在什么地方被阻塞时间最长
### 性能测量
步骤：</br>
* 测任务所花费时间
* 对测量结果进行统计排序，优先级高的放在前面</br>

工具：</br>
New Relic、xhprof、IFp

### 性能优化
优化原则：</br>
* 优化占总响应时间比重多的查询（>5%）
* 优化收益大于成本
* 优化离差指数高（方差均值：查询执行时间波动大）的查询

## MySQL查询性能
### 分析多条查询
#### 慢查询日志
优点：开销低、精度高</br>
缺点：消耗大量磁盘，要定时log rotate</br>
一般用法是将慢查询日志文件作为参数传递给 pt-query-digest 进行分析
#### 抓包
使用 tcpdump 

### 分析单条查询
#### SHOW PROFILE
启动 profiling 以后，profile 会记录服务器上执行的所有语句并测量其耗费时间、以及记录其他一些执行状态相关信息。
```
# 默认关闭，需启动，每次重新登录数据库都要重启
MariaDB [sakila]> SET profiling=1;
```
实例：
MySQL Sakila样本数据库查询
```
MariaDB [sakila]> select * from nicer_but_slower_film_list
997 rows in set (0.14 sec)

# 查看所有查询花费的时间
MariaDB [sakila]> SHOW PROFILES;
+----------+------------+-------------------------------------------+
| Query_ID | Duration   | Query                                     |
+----------+------------+-------------------------------------------+
|        1 | 0.00021079 | show tables                               |
|        2 | 0.00269660 | show databases                            |
|        3 | 0.00047102 | SELECT DATABASE()                         |
|        4 | 0.00128614 | show databases                            |
|        5 | 0.00143950 | show tables                               |
|        6 | 0.00062660 | show tables                               |
|        7 | 0.14167750 | select * from  nicer_but_slower_film_list |
+----------+------------+-------------------------------------------+
7 rows in set (0.00 sec)

# 查看指定查询的每个步骤花费时间（按执行顺序）
MariaDB [sakila]> SHOW PROFILE FOR QUERY 7;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000141 |
| checking permissions | 0.000024 |
| Opening tables       | 0.000867 |
| After opening tables | 0.000207 |
| System lock          | 0.000185 |
| Table lock           | 0.000021 |
| After table lock     | 0.000031 |
| init                 | 0.000028 |
| checking permissions | 0.000019 |
| checking permissions | 0.000013 |
| checking permissions | 0.000013 |
| checking permissions | 0.000013 |
| checking permissions | 0.000235 |
| optimizing           | 0.000020 |
| optimizing           | 0.000094 |
| statistics           | 0.000196 |
| preparing            | 0.000092 |
| statistics           | 0.000083 |
| preparing            | 0.000025 |
| executing            | 0.000017 |
| Sending data         | 0.002529 |
| executing            | 0.000049 |
| Creating tmp table   | 0.001617 |
| Copying to tmp table | 0.099986 |
| Sorting result       | 0.008461 |
| Sending data         | 0.022599 |
| removing tmp table   | 0.000318 |
| Sending data         | 0.003294 |
| end                  | 0.000026 |
| query end            | 0.000010 |
| closing tables       | 0.000005 |
| removing tmp table   | 0.000349 |
| closing tables       | 0.000033 |
| freeing items        | 0.000019 |
| removing tmp table   | 0.000010 |
| freeing items        | 0.000013 |
| updating status      | 0.000023 |
| cleaning up          | 0.000009 |
+----------------------+----------+
38 rows in set (0.01 sec)
```
可以看到最耗费时间的步骤是 Copying to tmp table | 0.099986 将数据复制到临时表

#### SHOW STATUS
SHOW STATUS 返回一些计数器，反应某些活动的频繁程度 </br>
SHOW STATUS 提供服务器状态信息，可以设置记录的时间节点，此信息也可以使用mysqladmin extended-status命令获得。
```
SHOW [GLOBAL | SESSION] STATUS [LIKE 'pattern'];
```
```
# 注意使用前要先刷新一下才能统计单条查询的数据
MariaDB [sakila]> FLUSH STATUS;
Query OK, 0 rows affected (0.00 sec)

MariaDB [sakila]> SELECT * FROM nicer_but_slower_film_list;  

MariaDB [sakila]> SHOW STATUS WHERE Variable_name LIKE 'Handler%' OR Variable_name LIKE 'Created%' ;
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Created_tmp_disk_tables    | 2     |
| Created_tmp_files          | 0     |
| Created_tmp_tables         | 3     |
| Handler_commit             | 1     |
| Handler_delete             | 0     |
| Handler_discover           | 0     |
| Handler_icp_attempts       | 0     |
| Handler_icp_match          | 0     |
| Handler_mrr_init           | 0     |
| Handler_mrr_key_refills    | 0     |
| Handler_mrr_rowid_refills  | 0     |
| Handler_prepare            | 0     |
| Handler_read_first         | 0     |
| Handler_read_key           | 7477  |
| Handler_read_last          | 0     |
| Handler_read_next          | 6462  |
| Handler_read_prev          | 0     |
| Handler_read_rnd           | 5462  |
| Handler_read_rnd_deleted   | 0     |
| Handler_read_rnd_next      | 6478  |
| Handler_rollback           | 0     |
| Handler_savepoint          | 0     |
| Handler_savepoint_rollback | 0     |
| Handler_tmp_update         | 0     |
| Handler_tmp_write          | 6459  |
| Handler_update             | 0     |
| Handler_write              | 0     |
+----------------------------+-------+
27 rows in set (0.00 sec)

```

## 服务器性能
### show global status
show global status  用于诊断服务器层面的问题，收集全局数据
```
MariaDB [(none)]> show global status where Variable_name='Threads_connected' or Variable_name='Threads_running'
    -> or Variable_name='Queries';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Queries           | 1320  |
| Threads_connected | 3     |
| Threads_running   | 1     |
+-------------------+-------+
3 rows in set (0.01 sec)
```
使用 mysqladmin 来进行 global status 的动态输出
```
$ mysqladmin -usharonli -p ext -i1
```
结合 awk 来进行分析：输出每秒查询数、当前打开的连接的数量、当前正在执行查询的线程数
```
# 每秒输出一次
$ mysqladmin -usharonli -p ext -i1 |awk '/Queries/{q=$4-qp;qp=$4}/Threads_connected/{tc=$4}/Threads_running/{printf "%5d %5d %5d \n",q,tc,$4}'
Enter password:
 1419     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1
    1     4     1

```
### show processlist
SHOW PROCESSLIST显示正在运行的线程。您也可以使用mysqladmin processlist语句得到此信息。如果您有SUPER权限，您可以看到所有线程，否则，您只能看到您自己的线程（也就是，与您正在使用的MySQL账户相关的线程）。
```
SHOW [FULL] PROCESSLIST;
```
需要关心的是 State 中的内容
```
$ mysql -usharonli -p -e 'show processlist'
Enter password:
+----+----------+-----------+------+---------+------+-------+------------------+----------+
| Id | User     | Host      | db   | Command | Time | State | Info             | Progress |
+----+----------+-----------+------+---------+------+-------+------------------+----------+
| 18 | sharonli | localhost | NULL | Sleep   | 1675 |       | NULL             |    0.000 |
| 21 | sharonli | localhost | NULL | Sleep   | 1568 |       | NULL             |    0.000 |
| 26 | sharonli | localhost | NULL | Query   |    0 | NULL  | show processlist |    0.000 |
+----+----------+-----------+------+---------+------+-------+------------------+----------+
```
### show innodb status
