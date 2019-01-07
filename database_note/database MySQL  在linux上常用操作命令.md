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
