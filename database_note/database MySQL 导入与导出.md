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
## 导出
```
SELECT 列1，列2 INTO OUTFILE '文件路径和文件名' FROM 表名字;
```
