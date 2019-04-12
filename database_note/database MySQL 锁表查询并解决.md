# MySQL 锁表查询并解决
## 锁表查询
查看数据库隔离级别
```
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set
```
