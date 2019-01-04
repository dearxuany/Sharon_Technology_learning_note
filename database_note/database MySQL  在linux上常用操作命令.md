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
