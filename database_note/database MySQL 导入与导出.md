# database MySQL 导入与导出
## 导入
###  SQL 文件导入
SQL 文件导入以SQL脚本形式执行SQL语句导入 \*.SQL 文件中的数据，一般用于建表。可以实现多种操作，包括删除，更新，新增，甚至对数据库的重建。
```
source *.sql  # 必须写sql文件的绝对路径
```
### 纯数据文件导入
可以把一个文件里的数据保存进一张表，该文件中将包含与数据表字段相对应的多条数据，这样可以快速导入大量数据。</br>
```
# 包含数据的txt文件
6	Alex	26	3000	123456	dpt1
7	Ken	27	3500	654321	dpt1
8	Rick	24	3500	987654	dpt3
9	Joe	31	3600	100129	dpt2
10	Mike	23	3400	110110	dpt1
11	Jim	35	3000	100861	dpt4
12	Mary	21	3000	100101	dpt2
```
由于导入导出大量数据都属于敏感操作，根据 mysql 的安全策略，导入导出的文件都必须在指定的路径下进行，在 mysql 终端中查看路径变量：
```
mysql> SHOW variables LIKE '%secure%';
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| require_secure_transport | OFF                   |
| secure_auth              | ON                    |
| secure_file_priv         | /var/lib/mysql-files/ |
+--------------------------+-----------------------+
3 rows in set (0.01 sec)
```
secure_file_priv 变量指定安全路径为 /var/lib/mysql-files/ ，要导入数据文件，需要将该文件移动到安全路径下。
```
$ sudo cp -a /home/shiyanlou/Desktop/SQL6 /var/lib/mysql-files 
```
导入 txt 中的数据
```
mysql> use mysql_shiyan;
Database changed
mysql> select * from employee;
+----+------+------+--------+--------+--------+
| id | name | age  | salary | phone  | in_dpt |
+----+------+------+--------+--------+--------+
|  1 | Tom  |   26 |   2500 | 119119 | dpt4   |
|  2 | Jack |   24 |   2500 | 120120 | dpt2   |
|  3 | Jobs | NULL |   3600 |  19283 | dpt2   |
|  4 | Tony | NULL |   3400 | 102938 | dpt3   |
|  5 | Rose |   22 |   2800 | 114114 | dpt3   |
+----+------+------+--------+--------+--------+
5 rows in set (0.00 sec)


mysql> LOAD DATA INFILE '/var/lib/mysql-files/SQL6/in.txt' INTO TABLE employee;
Query OK, 7 rows affected (0.02 sec)
Records: 7  Deleted: 0  Skipped: 0  Warnings: 0

mysql> select * from employee;
+----+------+------+--------+--------+--------+
| id | name | age  | salary | phone  | in_dpt |
+----+------+------+--------+--------+--------+
|  1 | Tom  |   26 |   2500 | 119119 | dpt4   |
|  2 | Jack |   24 |   2500 | 120120 | dpt2   |
|  3 | Jobs | NULL |   3600 |  19283 | dpt2   |
|  4 | Tony | NULL |   3400 | 102938 | dpt3   |
|  5 | Rose |   22 |   2800 | 114114 | dpt3   |
|  6 | Alex |   26 |   3000 | 123456 | dpt1   |
|  7 | Ken  |   27 |   3500 | 654321 | dpt1   |
|  8 | Rick |   24 |   3500 | 987654 | dpt3   |
|  9 | Joe  |   31 |   3600 | 100129 | dpt2   |
| 10 | Mike |   23 |   3400 | 110110 | dpt1   |
| 11 | Jim  |   35 |   3000 | 100861 | dpt4   |
| 12 | Mary |   21 |   3000 | 100101 | dpt2   |
+----+------+------+--------+--------+--------+
12 rows in set (0.00 sec)
```
数据量大时，一般会先建表，然后将数据写在txt中，每行包含一个记录，用定位符(tab)把值分开，并且按照上面的 CREATE TABLE 语句中列出的次序依次填写数据。对于丢失的值(例如未知的性别，或仍然活着的动物的死亡日期)，你可以使用 \N（反斜线 + 字母N）表示该值属于 NULL。最后将其放在在 /var/lib/mysql-files/ 中并导入到数据库中。</br>
注意：要关注linux和win上txt编辑器的换行和空格问题！</br>

### 关于 secure_file_priv
secure_file_priv 设置
* secure_file_priv为null    表示不允许导入导出
* secure_file_priv指定文件夹时，表示mysql的导入导出只能发生在指定的文件夹
* secure_file_priv没有设置时，则表示没有任何限制
```
# 此处表示没有设置指定安全路径，可以在任何路径进行导出导入

MariaDB [(none)]> SHOW variables LIKE '%secure%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_auth      | OFF   |
| secure_file_priv |       |
+------------------+-------+
2 rows in set (0.01 sec)
```
在任意文件夹尝试导入数据
```
4       Li      Lily    SCNU    Guangzhou
5       Chen    Rain    \N      Guangzhou
```
```
MariaDB [personalFinancialDB]> LOAD DATA INFILE '/home/sunnylinux/mysqltest/persons_data.txt' INTO TABLE persons;
ERROR 13 (HY000): Can't get stat of '/home/sunnylinux/mysqltest/persons_data.txt' (Errcode: 13)
```
貌似是权限问题，需要加个LOCAL参数：</br>
出于安全原因，当读取位于服务器中的文本文件时， 文件必须位于数据库目录中，或者是全体可读的。另外，要对服务器文件使用LOAD DATA INFILE，您必须拥有FILE权限。 如果指定了LOCAL，则文件会被客户主机上的客户端读取，并被发送到服务器。文件会被给予一个完整的路径名称，以指定确切的位置。</br>
```
MariaDB [personalFinancialDB]> LOAD DATA LOCAL INFILE '/home/sunnylinux/mysqltest/persons_data.txt' INTO TABLE persons;
Query OK, 3 rows affected, 11 warnings (0.40 sec)
Records: 4  Deleted: 0  Skipped: 1  Warnings: 11

MariaDB [personalFinancialDB]> SELECT * FROM persons;                           
+------------+----------+-----------+---------+-----------+
| ID_persons | LastName | FirstName | Address | City      |
+------------+----------+-----------+---------+-----------+
|          1 | Li       | Sharon    | NULL    | NULL      |
|          2 | Tang     | Jack      | NULL    | NULL      |
|          3 | Tang     | Ben       | NULL    | NULL      |
|          4 | Li       | Lily      | SCNU    | Guangzhou |
|          5 | Chen     | Rain      | NULL    | Guangzhou |
|          6 |          | NULL      | NULL    | NULL      |
|          7 |          | NULL      | NULL    | NULL      |
+------------+----------+-----------+---------+-----------+
7 rows in set (0.01 sec)

```
注意：txt中的空白行要注意删除，否则也会被mysql导入到数据库当中，例如上面的6、7行

## 导出
```
SELECT 列1，列2 INTO OUTFILE '文件路径和文件名' FROM 表名字;
```
注意权限问题，因为当前mysql账户没有文件夹的写权限
```
MariaDB [personalFinancialDB]> select * from persons where Firstname='Sharon' INTO OUTFILE '/home/sunnylinux/mysqltest/persons_sharon.txt';
ERROR 1 (HY000): Can't create/write to file '/home/sunnylinux/mysqltest/persons_sharon.txt' (Errcode: 13)
```
改到输出到/tmp就可以
```
MariaDB [personalFinancialDB]> select * from persons where Firstname='Sharon' INTO OUTFILE '/tmp/persons_sharon.txt';
Query OK, 1 row affected (0.37 sec)
```
参考：https://www.linuxidc.com/Linux/2012-02/55533.htm
