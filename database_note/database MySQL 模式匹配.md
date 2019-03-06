# database MySQL 模式匹配
## 标准SQL匹配
* 在 MySQL中，SQL的模式默认是忽略大小写的;</br>
* 允许使用 _ 匹配任何单个字符，% 匹配任意数目字符(包括零字符)；</br>
* 使用SQL模式时，不能使用=或!=；而应使用LIKE或NOT LIKE比较操作符。</br>
```
MariaDB [personalFinancialDB]> select * from persons WHERE FirstName='S%';
Empty set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName LIKE '%n';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          1 | Li       | Sharon    | NULL    | NULL      | 1993-02-26 |
|          3 | Tang     | Ben       | NULL    | NULL      | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.03 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName LIKE 'J%';
+------------+----------+-----------+---------+------+-------+
| ID_persons | LastName | FirstName | Address | City | Birth |
+------------+----------+-----------+---------+------+-------+
|          2 | Tang     | Jack      | NULL    | NULL | NULL  |
+------------+----------+-----------+---------+------+-------+
1 row in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName LIKE '%a%';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          1 | Li       | Sharon    | NULL    | NULL      | 1993-02-26 |
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName LIKE '____';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          4 | Li       | Lily      | SCNU    | Guangzhou | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName LIKE '__i_';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
1 row in set (0.00 sec)
```
