# database MySQL 模式匹配
## 标准SQL匹配
* 在 MySQL中，SQL的模式默认是忽略大小写的;</br>
* 允许使用 _ 匹配任何单个字符，% 匹配任意数目字符(包括零字符)；</br>
* 使用SQL模式时，不能使用=或!=；而应使用LIKE或NOT LIKE比较操作符，精准匹配。</br>
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
## 扩展正则表达式
* 使用REGEXP和NOT REGEXP操作符（或RLIKE和NOT RLIKE）；</br>
* 使用BINARY关键字加在REGEXP后面，使其中一个字符串变为二进制字符串，以强制区分大小写；</br>
* 为了定位一个模式以便它必须匹配被测试值的开始或结尾，在模式开始处使用“^”或在模式的结尾用“$”；</br>
* 使用.表示单个字符，.{n}表示某个字符重复n次，\[0-9\]表示0到9其中一个。</br>
```
MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP 'n$';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          1 | Li       | Sharon    | NULL    | NULL      | 1993-02-26 |
|          3 | Tang     | Ben       | NULL    | NULL      | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.03 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP '^S';
+------------+----------+-----------+---------+------+------------+
| ID_persons | LastName | FirstName | Address | City | Birth      |
+------------+----------+-----------+---------+------+------------+
|          1 | Li       | Sharon    | NULL    | NULL | 1993-02-26 |
+------------+----------+-----------+---------+------+------------+
1 row in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP 'a';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          1 | Li       | Sharon    | NULL    | NULL      | 1993-02-26 |
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP '^....$';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          4 | Li       | Lily      | SCNU    | Guangzhou | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP '^.{4}$';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          4 | Li       | Lily      | SCNU    | Guangzhou | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
3 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from persons WHERE FirstName REGEXP '^..i.$';
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
1 row in set (0.00 sec)

```



