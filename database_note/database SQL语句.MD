# SQL基础语句
SQL特性： 
* 对大小写不敏感，SELECT 与 select 是相同的，新建的数据库时名称要用小写，语句用大写会比较好分辨;
* 对空格不敏感，可以换行，增加可读性；
* 多条SQL语句必须用分号；分隔，单条语句部分RDBMS不需要用分号分隔;
* 复合语句，用括号（）括起内部优先执行的语句；
* 不同数据库中的SQL语法会有略微不同（此处主要记录mySQL）。</br>

SQL语句的分类：</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/database_note_images/sql-command.png)
* DDL data definition language 数据定义语言</br>
用于定义或改变表（TABLE）的结构、数据类型、视图和索引的创建和撤消操作，他们大多在建立表时使用，DDL不需要commit
```
CREATE DATABASE - 创建新数据库
ALTER DATABASE - 修改数据库
CREATE TABLE - 创建新表
ALTER TABLE - 变更（改变）数据库表
DROP TABLE - 删除表
TRUNCATE TABLE - 清空表中数据（不能用于参与了索引视图的表）
CREATE INDEX - 创建索引（搜索键）
DROP INDEX - 删除索引
COMMENT - 注释
RENAME
```
* DML data manipulation language 数据操作语言</br>
用于对已有表的数据进行增删改查操作，DML需要commit
```
SELECT - 从数据库中提取数据
UPDATE - 更新数据库中的数据
DELETE - 从数据库中删除数据
INSERT INTO - 向数据库中插入新数据
MERGE
CALL
EXPLAIN PLAN
LOCK TABLE - 锁
```
* DCL data control language 数据控制语言</br>
用于授权，角色控制等
```
GRANT - 授权
REVOKE - 取消授权
```
* TCL traction control language 事务控制语言</br>
```
COMMIT - 提交
SAVEPOINT - 设置保存点
ROLLBACK  - 回滚
SET TRANSACTION 设置当前事务的特性，它对后面的事务没有影响
```
注释
```
# 整行注释
-- 行内注释
/* 多行注释 */
```

## select 选择
选出需要的列
```
SELECT column_name1,column_2 FROM database_name.table_name；
```
提取整个表
```
SELECT * FROM database_name.table_name；
```
```
SELECT first_name,last_name FROM sakila.actor;
SELECT * FROM sakila.actor;
```

## SELECT DISTINCT 仅列出不同的值
返回唯一不同的值
```
SELECT DISTINCT column_name FROM database_name.table_name;
```
```
SELECT DISTINCT building FROM university.department;
```


## where 条件判断
先选取再筛选
```
SELECT column_name FROM WHERE column_name >= value；
```
直接筛选
```
SELECT * FROM WHERE column_name <> value；
```
value：字符串用单引号括起，数字不用</br>

逻辑运算符：
```
=	等于
<=>  空值安全等号
<>	不等于
>	大于
<	小于
>=	大于等于
<=	小于等于
BETWEEN	在某个范围内
LIKE	搜索某种模式
```
```
SELECT * FROM university.department WHERE budget = 6000;
SELECT budget FROM university.department WHERE budget = 6000;
SELECT * FROM university.department WHERE building = 'waston';
```
对于NULL值的处理：</br>
选取表中不为空的数据，使用 IS NOT NULL；</br>
选取表中为空的数据，使用 IS NULL；</br>
在GROUP BY中，两个NULL值被视为相同等价的。</br>

```
select lastName,FirstName,City FROM personalFinancialDB.persons
WHERE city IS NOT NULL
```
```
select lastName,FirstName,City FROM personalFinancialDB.persons
WHERE city IS NULL
```
NULL值比较
```
MariaDB [(none)]> select null=null,null<=>null,1=1,1<=>1,1=null,1<=>null,0=null,0<=>null;
+-----------+-------------+-----+-------+--------+----------+--------+----------+
| null=null | null<=>null | 1=1 | 1<=>1 | 1=null | 1<=>null | 0=null | 0<=>null |
+-----------+-------------+-----+-------+--------+----------+--------+----------+
|      NULL |           1 |   1 |     1 |   NULL |        0 |   NULL |        0 |
+-----------+-------------+-----+-------+--------+----------+--------+----------+
1 row in set (0.01 sec)
```


## and和or 复合条件判断
用于where后面的符合条件筛选，where后面可用括号括起
```
SELECT * FROM database_name.table_name WHERE column_name1 >= value AND column_name2 = value ; 
SELECT * FROM database_name.table_name 
WHERE (column_name1 >= value AND column_name2 = value) OR column_name3 <= value;
```
```
SELECT * FROM university.department WHERE building = 'waston' AND budget < 6000;
SELECT * FROM university.department WHERE building = 'waston' OR budget < 6000;
```

## ORDER BY 排序
ORDER BY 语句用于根据指定的列对结果集进行排序；</br>
默认按照升序对记录进行排序，按照降序对记录进行排序；</br>
可以使用 DESC 关键字，升序排列关键字ASC；</br>
可以写多个排序条件，用逗号隔开，优先满足前面条件，再按后面条件排序。</br>
```
SELECT * FROM database_name.table_name ORDER BY column_name;
SELECT * FROM database_name.table_name ORDER BY column_name DESC;
```
```
SELECT * FROM university.department ORDER BY budget;
SELECT * FROM university.department ORDER BY dept_name;
SELECT * FROM university.department ORDER BY dept_name DESC;
SELECT * FROM university.department ORDER BY dept_name DESC,budget ASC;
```
## INSERT 插入
插入一整行数据，前提是插入value个数和table的列数是一样的
```
INSERT INTO database_name.table_name VALUES (value1,value2,value3...)
```
给指定行插入数据，没有指定的列的值为NULL
```
INSERT INTO database_name.table_name (column1,column3) 
VALUES (value1,value3)
```
(注意插入是没有输出的！)
```
insert into university.department values ('math','harry',3000);
insert into university.department (building,budget) 
values ('iron',6000); 
```
同时插入多行数据
```
INSERT INTO dbtest.orders
VALUES (1,77895,3),(2,44678,3),(3,22456,1),(4,24562,1),(5,34764,65);
```
## UPDATE和SET 修改value
修改表中数据，where后面是选择要修改的位置的条件（修改 column_name2=value处的column_name1的值）
```
UPDATE database_name.table_name SET column_name1=new_value WHERE column_name2=value
```
更新某行的若干列
```
UPDATE database_name.table_name SET column_name1=new_value,column_name2=new_value2 WHERE column_name3=value
```
```
update university.department set budget=5600 where dept_name='math'
update university.department set budget=7000,building='luke' where dept_name='math'
```
注意：UPDATE 一定要配合 where 用，否则会改变该column的所有值！！
## DELETE 删除某行
删除column_name=value处的那些行（会删除多行）</br>
WHERE 子句规定哪条记录或者哪些记录需要删除。如果省略 WHERE 子句，所有的记录都将被删除！</br>
```
DELETE FROM database_name.table_name WHERE column_name=value
```
不删除表的前提下删除所有行（删除表中的数据），表结构、属性、索引将保持不变，不释放空间。</br>
delete 命令删除的数据将存储在系统回滚段中，需要的时候，数据可以回滚恢复。</br>
```
DELETE FROM database_name.table_name
```
```
delete from university.department where budget='5000'  #删不掉
delete from university.department  #删不掉
```


# SQL进阶语句
## LIMIT 和 OFFSET
规定要返回的记录的数目(mySQL语法)
```
SELECT column_name(s)
FROM table_name
LIMIT number
```
返回前20行
```
SELECT * FROM sakila.actor LIMIT 20;
```
由第5行开始，返回后面6行数据。
SQL中检索行号是由0开始，即4代表第5行。
```
SELECT * FROM sakila.actor
LIMIT 6 OFFSET 4;
```
也可简写，前面数字对应OFFSET起始行号，后面数字代表LIMIT显示行数。
```
SELECT * FROM sakila.actor
LIMIT 4,6;
```

## LIKE
LIKE 操作符用于在 WHERE 子句中搜索列中的指定格式
```
SELECT column_name(s)
FROM table_name
WHERE column LIKE sth;
```
通配符%，搜索last_name以g开头的行
```
SELECT * FROM sakila.actor WHERE last_name LIKE 'g%';
```
搜索last_name以g结束的行
```
SELECT * FROM sakila.actor WHERE last_name LIKE '%g';
```
搜索last_name中间有特定字符的行
```
SELECT * FROM sakila.actor WHERE last_name LIKE '%oo%';
```
SQL 通配符:SQL 通配符必须与 LIKE 运算符一起使用。
```
%                             替代一个或多个字符
_                             仅替代一个字符
[charlist]                    字符列中的任何单一字符
[^charlist]或者[!charlist]    不在字符列中的任何单一字符
```
```
SELECT * FROM sakila.actor WHERE last_name LIKE '_K%';      第二个字母为k的姓
SELECT * FROM sakila.actor WHERE last_name LIKE '[DB]%';    选以D或B开头的姓（没有成功）
SELECT * FROM sakila.actor WHERE last_name LIKE '[！DB]%';  选不以D或B开头的姓（没有成功）
```
## IN
选取多个值为value的行
```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```
```
SELECT * FROM sakila.actor WHERE last_name IN ('CHASE','DAVIS');
```
## BETWEEN
选取介于两个值之间的数据范围，这些值可以是数值、文本或者日期。
```
SELECT column_name(s)
FROM table_name
WHERE column_name
BETWEEN value1 AND value2
```
范围50（包括）到100（包括）
```
SELECT * FROM sakila.actor WHERE actor_id BETWEEN 50 AND 100;
```
当value为字符串时，按字母顺序排序后选取,但是输出结果只是选取了在范围之间的值，并没有排序的。</br>
mySQL范围A（包括）到C（不包括），不同数据库之前有差异。
```
SELECT * FROM sakila.actor WHERE first_name BETWEEN 'A%' AND 'C%';
```
选范围之外
```
SELECT * FROM sakila.actor WHERE first_name NOT BETWEEN 'A%' AND 'C%';
```
注意：数字和字符串的处理
```
MariaDB [(none)]> select 1 between 2 and 3;
+-------------------+
| 1 between 2 and 3 |
+-------------------+
|                 0 |
+-------------------+
1 row in set (0.04 sec)

MariaDB [(none)]> select 2 between 1 and '9';
+---------------------+
| 2 between 1 and '9' |
+---------------------+
|                   1 |
+---------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select b between a and c;
ERROR 1054 (42S22): Unknown column 'b' in 'field list'
MariaDB [(none)]> select 'b' between 'a' and 'c';
+-------------------------+
| 'b' between 'a' and 'c' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set (0.00 sec)
```
## ALIAS
为列名称和表名称指定别名Alias，AS的位置紧跟要定别名的表或列名，别名使查询程序更易阅读和书写。</br>
表
```
SELECT column_name(s)
FROM table_name
AS alias_name
```
将一个含有演员信息的表 sakila.actor指定别名为a，将另外一个有电影参演演员信息的表sakila.film_actor指定别名为fa，查演员ID为1的演员的名称和他参演过的电影的ID。
```
SELECT a.first_name, a.last_name, fa.film_id 
FROM sakila.actor AS a, sakila.film_actor AS fa 
WHERE a.actor_id=1 AND fa.actor_id=1;
```

列
```
SELECT column_name AS alias_name
FROM table_name
```
```
SELECT first_name AS fname FROM sakila.actor
```

## JOIN
join 用于根据两个或多个表中的列之间的关系，从这些表中查询数据。</br>
数据库中的表可通过键将彼此联系起来。</br>
主键（Primary Key）是一个列，在这个列中的每一行的值都是唯一的。在表中，每个主键的值都是唯一的。</br>
这样做的目的是在不重复每个表中的所有数据的情况下，把表间的数据交叉捆绑在一起。</br>
</br>
实例：有三个表，分别是actor、file_actor、file，查询每个演员各自演过的电影。</br>
```
SELECT a.first_name, a.last_name, f.title 
FROM sakila.actor AS a, sakila.film_actor AS fa, sakila.film AS f
WHERE a.actor_id=fa.actor_id AND fa.film_id=f.film_id;
```
使用join来获取数据
```
SELECT a.first_name, a.last_name, f.title 
FROM sakila.actor AS a 
INNER JOIN sakila.film_actor AS fa
ON a.actor_id=fa.actor_id
INNER JOIN sakila.film AS f
ON fa.film_id=f.film_id;
```
不同的JOIN
* JOIN: INNER JOIN 与 JOIN 是相同的，如果表中有至少一个匹配，则返回行
* LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行
* RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
* FULL JOIN: 只要其中一个表中存在匹配，就返回行

## LEFT JOIN
LEFT JOIN 关键字会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行。
```
SELECT column_name(s)
FROM table_name1
LEFT JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```
实例：</br>
actor、file_actor，查询每个演员各自演过的电影的电影编号，同时就算这个演员没演过电影也返回他的信息。</br>
在前面的表格插一个没演过电影的人的信息：</br>
```
INSERT INTO sakila.actor VALUES (201,'SHARON','LI','2015-08-03 18:20:11')
```
因为数据太多会显示不全，所以加最后一句限制范围
```
SELECT a.first_name, a.last_name, fa.film_id
FROM sakila.actor AS a 
LEFT JOIN sakila.film_actor AS fa
ON a.actor_id=fa.actor_id
WHERE a.actor_id BETWEEN 199 AND 201
```

## RIGHT JOIN 
RIGHT JOIN 关键字会右表 (table_name2) 那里返回所有的行，即使在左表 (table_name1) 中没有匹配的行。
```
SELECT column_name(s)
FROM table_name1
RIGHT JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```
实例：</br>
actor、file_actor，查询每个演员各自演过的电影的电影编号，同时就算这个演员没演过电影也返回他的信息。这次把两个表的顺序调转一下。</br>
```
SELECT sakila.actor.first_name,sakila.actor.last_name,sakila.film_actor.film_id
FROM sakila.film_actor
RIGHT JOIN sakila.actor
ON sakila.film_actor.actor_id = sakila.actor.actor_id
WHERE sakila.actor.actor_id BETWEEN 199 AND 201
```

## FULL JOIN
列出两个表里面的所有数据，不管有没有对应的匹配。
```
SELECT column_name(s)
FROM table_name1
FULL JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```

## UNION 和 UNION ALL
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。</br>
注意：</br>
两句SELECT后面必须有相同数量的column；</br>
要整合的column的顺序必须一一对应；</br>
column内数据的数据类型必须相同；</br>
UNION会删去重复的数据，仅显示一条；</br>
如果允许出现重复的值，可使用UNION ALL。

```
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
```
整合两个表格中的城市数据
```
SELECT city FROM sakila.city
union
SELECT city FROM dbtest.persons
```
```
SELECT city FROM sakila.city
union all
SELECT city FROM dbtest.persons
```

##  INSERT INTO SELECT
INSERT INTO SELECT 语句从一个表复制数据，然后把数据插入到一个已存在的表中。目标表中任何已存在的行都不会受影响。
（注意：mySQL比较特殊，用的INSERT INTO SELECT，其他数据库的语法是SELECT INTO）
</br>
将所有列插入新表，新表必须是已存在的。
```
INSERT INTO table2
SELECT * FROM table1;
```
将部分列插入新表
```
INSERT INTO table2
(column_name(s))
SELECT column_name(s)
FROM table1;
```

## CREATE INDEX
CREATE INDEX 语句用于在表中创建索引，索引使数据库应用程序可以更快地查找数据。</br>
注意：</br>
更新一个包含索引的表需要比更新一个没有索引的表更多的时间，这是由于索引本身也需要更新。</br>
因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。</br>

CREATE INDEX 语句允许使用重复的值：
```
CREATE INDEX index_name
ON table_name (column_name)
```
CREATE UNIQUE INDEX 语句创建一个唯一的索引，唯一的索引意味着两个行不能拥有相同的索引值：
```
CREATE UNIQUE INDEX index_name
ON table_name (column_name)
```
实例：</br>
给多个列添加索引，以便直接查询全名</br>
```
CREATE INDEX personFullnameIndex
ON personalFinancialDB.persons (LastName,FirstName)
```


## CREATE DATABASE
CREATE DATABASE 用于创建数据库。
```
CREATE DATABASE database_name
```
新建一个名为personalFinancialDB的数据库
```
CREATE DATABASE personalFinancialDB;
```

## CREATE TABLE
CREATE TABLE 语句用于创建数据库中的表。
```
CREATE TABLE new_table_name
(
column_name1 data_type(size),
column_name2 data_type(size),
column_name3 data_type(size),
)
```

实例：</br>
在上面创建的数据库中，创建一个名为persons的空白表。
其中主键是ID，ID的号码在插入新数据时自动增加，而不需要额外插入（使用关键词 AUTO_INCREMENT）。
```
CREATE TABLE personalFinancialDB.persons
(
ID_persons int NOT NULL AUTO_INCREMENT,     -- ID自动增加，不能为空
LastName varchar(255) NOT NULL,     -- 姓不能为空
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (ID_persons)                    -- 将ID设为主键
)
```
尝试插入数据
```
insert into personalFinancialDB.persons (LastName,FirstName)
VALUES ('Li','Sharon')
```
ID_persons自动增加为1，插入上面的values，其他位置为null。

### 数据类型 data type
不同数据库对数据类型的定义不一样，使用前注意查询官方文档！！
* 仅容纳整数,在括号内规定数字的最大位数：</br>
integer(size)</br>
int(size)</br>
smallint(size)</br>
tinyint(size)</br>

* 容纳带有小数的数字, "size" 规定数字的最大位数，"d" 规定小数点右侧的最大位数：</br>
decimal(size,d)</br>
numeric(size,d)</br>

* 容纳固定长度的字符串（可容纳字母、数字以及特殊字符），在括号中规定字符串的长度：</br>
char(size)</br>

* 容纳可变长度的字符串（可容纳字母、数字以及特殊的字符），在括号中规定字符串的最大长度:</br>
varchar(size)</br>

* 容纳日期：</br>
mySQL: DATETIME、TIMESTAMP、DATE、TIME、YEAR</br>
```
DATE()        日期，格式：YYYY-MM-DD，支持的范围是从 '1000-01-01' 到 '9999-12-31'
DATETIME()    日期和时间的组合，格式：YYYY-MM-DD HH:MM:SS，支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'
TIMESTAMP()   时间戳，TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的描述来存储，格式：YYYY-MM-DD HH:MM:SS，支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC
TIME()        时间，格式：HH:MM:SS，支持的范围是从 '-838:59:59' 到 '838:59:59'
YEAR()        2 位或 4 位格式的年，4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069
```

### 约束 Constraints
约束用于限制加入表的数据的类型；</br>
可以在创建表时规定约束（通过 CREATE TABLE 语句,写在括号内），或者在表创建之后也可以（通过 ALTER TABLE 语句）。</br>
</br>
主要有以下几种约束：
* NOT NULL
* UNIQUE
* PRIMARY KEY
* FOREIGN KEY
* CHECK
* DEFAULT

#### NOT NULL
默认可接受NULL值，强制列不接受 NULL 值，如果不向字段添加值，就无法插入新记录或者更新记录。
```
ID_persons int NOT NULL AUTO_INCREMENT,
LastName varchar(255) NOT NULL,
```

#### UNIQUE
唯一标识数据库表中的每条记录，为列或列集合提供了唯一性的保证，每个表可以有多个 UNIQUE 约束。</br>
新建表时，将单个列创建UNIQUE约束：
```
UNIQUE (column_name)
```
新建表时，给多个列创建UNIQUE约束：
```
CONSTRAINT constraint_name UNIQUE (column_name1，column_name2,...)
```

#### PRIMARY KEY
主键，唯一标识数据库表中的每条记录，不能包含 NULL 值，每个表都应该有一个主键，并且每个表只能有一个主键。</br>
单列定义的主键：
```
PRIMARY KEY (column_name)
```
使用多列作为主键：
```
CONSTRAINT constraint_name PRIMARY KEY (column_name1，column_name2,...)
```

#### FOREIGN KEY</br>
外键，一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。</br>
用于预防破坏表之间连接的动作，能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。</br>
```
ID_persons int,
FOREIGN KEY (ID_persons) REFERENCES persons(ID_persons)
```
实例：
创建支出表 expenditure（支出ID ID_expenditure，支出明细 expenseDetail，支出金额 expenseAmount，支出日期 expenseDate，支出用户 ID_persons）；</br>
将之前创建的persons表中的ID_persons设为支出表的外键，以表示“这是谁的支出”。
```
CREATE TABLE personalFinancialDB.expenditure
(
ID_expenditure int NOT NULL AUTO_INCREMENT,
expenseDetail varchar(255) NOT NULL,  
expenseAmount decimal(7,1),
expenseDate DATETIME,
ID_persons int,
PRIMARY KEY (ID_expenditure),
FOREIGN KEY (ID_persons) REFERENCES persons(ID_persons)
)
```
日期那里的datatype用了DATETIME，其格式为 YYYY-MM-DD HH:MM:SS。
尝试插入一条数据，日期处使用了函数 now()。
```
INSERT INTO personalFinancialDB.expenditure (expenseDetail,expenseAmount,expenseDate,ID_persons)
VALUES ('Mucha Milktea', 2.7, now(),1)
```

#### CHECK</br>
CHECK 用于限制列中的值的范围。</br>
如果对单个列定义 CHECK 约束，那么该列只允许特定的值。</br>
如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。</br>
</br>
新建表时，在括号内创建CHECK约束：
```
CHECK (column_name>0)
```
同时给约束起名字：
```
CONSTRAINT constraint_name CHECK (column_name1>0 AND column_name3='sth')
```

#### DEFAULT</br>
插入默认值，如果没有规定其他的值，那么会将默认值添加到所有的新记录。</br>
在新建表时，在datatype后面添加DEFAULTA约束，default_value为要设置的默认值：
```
column_name datatype DEFAULT default_value
```
如果插入的默认值需要函数参与，则可以改为以下样子：
```
column_name date DEFAULT NOW()
```
此处默认值为，当前系统时间。</br>

## ALTER TABLE
ALTER TABLE 用于在已有的表，如修改约束，添加、修改或删除列等。

### 修改已有表的约束
通用格式
```
ALTER TABLE table_name
ADD 约束类型 （约束内容）
```
改名的约束
```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name
约束类型 （约束内容）
```
输出约束
```
ALTER TABLE table_name
DROP 约束类型 constraint_name
```

#### UNIQUE</br>
给已有表创建单列 UNIQUE 约束：
```
ALTER TABLE table_name
ADD UNIQUE (column_name)
```
如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束：
```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name 
UNIQUE (column_name1，column_name2,...)
```
删除已有约束：
```
ALTER TABLE table_name
DROP INDEX constraint_name
```
实例：</br>
给上面创建的表的ID和LastName两列创建UNIQUE约束
```
ALTER TABLE personalFinancialDB.persons
ADD CONSTRAINT uqPersonID UNIQUE (ID,LastName)
```
删除UNIQUE约束
```
ALTER TABLE personalFinancialDB.persons
DROP INDEX uqPersonID
```

#### PRIMARY KEY
用 ALTER TABLE 语句添加主键前提： 表在新建的时，该column必须有not null的约束。
创建已有表的单列主键：
```
ALTER TABLE table_name
ADD PRIMARY KEY (column_name)
```
创建已有表的多列主键：
```
ALTER TABLE table_name
ADD CONSTRAINT constraint_name 
PRIMARY KEY (column_name1，column_name2,...)
```
撤销主键：
```
ALTER TABLE table_name
DROP primary key
```

#### FOREIGN KEY
给已有表设置FOREIGN KEY，前提是新建表时已新建外键用的column：
```
ALTER TABLE table_name2
ADD FOREIGN KEY (column_name)
REFERENCES table_name1(column_name)
```
也可给 FOREIGN KEY 约束改名：
```
ALTER TABLE  table_name2
ADD CONSTRAINT constraint_name
FOREIGN KEY (column_name)
REFERENCES table_name1(column_name)
```
撤销 FOREIGN KEY 约束：
```
ALTER TABLE table_name
DROP FOREIGN KEY constraint_name
```

#### CHECK
给已有表创建CHECK约束：
```
ALTER TABLE personalFinancialDB.expenditure
ADD CHECK (expenseAmount>0)
```
删除CHECK约束：
```
ALTER TABLE table_name
DROP CHECK constraint_name
```

#### DEFAULT
给已有表的已有列创建默认值：
```
ALTER TABLE table_name
ALTER column_name SET DEFAULT default_value
```
注意：</br>
使用函数来生成默认值貌似是不能在新建表之后再更改的，所以最好新建表的时候就设置好，不然就可能需要像修改列名那样改了，不能像其他约束那样修改。</br>
</br>
删除默认值：
```
ALTER TABLE table_name
ALTER column_name DROP DEFAULT
```

### 添加、删除、修改列
#### 修改列
##### 改列名
改列名的同时注意把后面的datatype也填好，不然会出错。
```
ALTER TABLE 表名字 CHANGE 原列名 新列名 数据类型 约束;
```
```
ALTER TABLE table_name 
CHANGE COLUMN old_column_name new_column_name datatype
```
实例：
```
ALTER TABLE personalFinancialDB.persons 
CHANGE COLUMN ID ID_persons int NOT NULL AUTO_INCREMENT
```
如果修改的是表的主键，注意添加后面的 NOT NULL AUTO_INCREMENT ，不然插入数据的时候会报错。

##### 改列的数据类型
mySQL语法中，改列名和改数据类型使用的语法是相同的，仅是修改的位置不同。
```
ALTER TABLE table_name
CHANGE COLUMN column_name column new_datatype
```
实例：
```
ALTER TABLE expenditure
CHANGE COLUMN expenseDate expenseDate DATE
```
当原列名和新列名相同的时候，指定新的数据类型或约束，就可以用于修改数据类型或约束。需要注意的是，修改数据类型可能会导致数据丢失，所以要慎重使用。
#### 添加列
```
ALTER TABLE 表名字 ADD COLUMN 列名字 数据类型 约束;
或：
ALTER TABLE 表名字 ADD 列名字 数据类型 约束;
```
默认添加列在表最后
```
ALTER TABLE table_name
ADD column_name datatype
```
实例：
```
# 在age后面新增体重信息，在第一列新增一列则将AFTER column_name 改为 FIRST
mysql> ALTER table employee ADD weight INT(4) DEFAULT 120 AFTER age;
```


#### 删除列
```
ALTER TABLE table_name 
DROP COLUMN column_name
```
实例：
```
ALTER TABLE university.department
DROP COLUMN budget
```

## DROP
删除索引(mySQL)
```
ALTER TABLE table_name 
DROP INDEX index_name
```
删除表
```
DROP TABLE table_name
```
删除数据库
```
DROP DATABASE database_name
```

## TRUNCATE TABLE 
TRUNCATE TABLE 仅仅需要除去表内的数据，但并不删除表本身。</br>
即是仅清空数据内容，表和表中的column还是存在的，不会被清空。
```
TRUNCATE TABLE table_name
```

# 字段与函数
## 字段
字段：</br>
在执行查询操作时，由函数新建出来的column并不真实存在于原始数据表中，用的时候才会自动生成，这个被生成的column被称为 “字段 field”，也通常被称为“计算字段”。</br>
支持使用内建函数和一般的算术运算来生成特定的算术结果，也可由内建函数或一些通配符来生成“拼接字段”，而 MySQL 貌似仅能通过函数来生成。</br>
</br>
注意：</br>
插入操作时使用函数处理插入的VALUE的格式时，这些格式是真实存在于原始数据表中的。</br>
插入的数据的类型、格式一定要和表创建时给该列设置的类型、格式匹配才可以。</br>
不过，查询的时候可以不直接显示原始数据的格式，而按实际需求来调整。</br>
### 算术计算
支持以下符号，并用()标示计算优先级:
```
+
-
*
/
```

实例：
有单价和产品数量，算出各个商品的消费总价
```
SELECT expenseDetail, unitPrice, expenseNumber,
unitPrice*expenseNumber AS expandedAmount
FROM expenditure
```

### 拼接
实例：</br>
输出每位顾客的ID和姓名以及在店内的总消费，MySQL中使用contac()函数。</br>
```
# 无格式
select sakila.customer.customer_id,sakila.customer.first_name, sakila.customer.last_name, SUM(sakila.payment.amount)
from sakila.payment,sakila.customer
WHERE sakila.payment.customer_id = sakila.customer.customer_id
GROUP BY sakila.customer.customer_id
```
将顾客的姓名合起来输出，并设置格式</br>
```
# 有格式
select sakila.customer.customer_id, 
concat(sakila.customer.first_name,' ',sakila.customer.last_name), SUM(sakila.payment.amount)
from sakila.payment,sakila.customer
WHERE sakila.payment.customer_id = sakila.customer.customer_id
GROUP BY sakila.customer.customer_id
```
也可以给这个拼接字段改名，改完之后列名就会变成 full_name
```
select sakila.customer.customer_id, 
concat(sakila.customer.first_name,' ',sakila.customer.last_name) AS full_name, 
SUM(sakila.payment.amount)
from sakila.payment,sakila.customer
WHERE sakila.payment.customer_id = sakila.customer.customer_id
GROUP BY sakila.customer.customer_id
```

## 日期函数
日期函数常用于插入数据时，让系统自动插入要求格式的日期。</br>
注意：如希望使查询简单且更易维护，不要在日期中使用时间部分！单用日期是无法查询到带时间的数据的。</br>
</br>
mySQL支持以下日期函数：
```
NOW()	        返回当前的日期和时间
CURDATE()	    返回当前的日期
CURTIME()	    返回当前的时间
DATE()	      提取日期或日期/时间表达式的日期部分
EXTRACT()	    返回日期/时间按的单独部分
DATE_ADD()	  给日期添加指定的时间间隔
DATE_SUB()	  从日期减去指定的时间间隔
DATEDIFF()	  返回两个日期之间的天数
DATE_FORMAT()	用不同的格式显示日期/时间
```
参数格式：
```
%a	缩写星期名
%b	缩写月名
%c	月，数值
%D	带有英文前缀的月中的天
%d	月的天，数值(00-31)
%e	月的天，数值(0-31)
%f	微秒
%H	小时 (00-23)
%h	小时 (01-12)
%I	小时 (01-12)
%i	分钟，数值(00-59)
%j	年的天 (001-366)
%k	小时 (0-23)
%l	小时 (1-12)
%M	月名
%m	月，数值(00-12)
%p	AM 或 PM
%r	时间，12-小时（hh:mm:ss AM 或 PM）
%S	秒(00-59)
%s	秒(00-59)
%T	时间, 24-小时 (hh:mm:ss)
%U	周 (00-53) 星期日是一周的第一天
%u	周 (00-53) 星期一是一周的第一天
%V	周 (01-53) 星期日是一周的第一天，与 %X 使用
%v	周 (01-53) 星期一是一周的第一天，与 %x 使用
%W	星期名
%w	周的天 （0=星期日, 6=星期六）
%X	年，其中的星期日是周的第一天，4 位，与 %V 使用
%x	年，其中的星期一是周的第一天，4 位，与 %v 使用
%Y	年，4 位
%y	年，2 位
```

实例：
```
select customer_id, SUM(amount), DATE_FORMAT(Now(),'%b %d %Y %h:%i %p')
from sakila.payment
GROUP BY customer_id
```
### 日期函数计算
#### 年龄计算
```
MariaDB [personalFinancialDB]> select * from persons;                           
+------------+----------+-----------+---------+-----------+------------+
| ID_persons | LastName | FirstName | Address | City      | Birth      |
+------------+----------+-----------+---------+-----------+------------+
|          1 | Li       | Sharon    | NULL    | NULL      | 1993-02-26 |
|          2 | Tang     | Jack      | NULL    | NULL      | NULL       |
|          3 | Tang     | Ben       | NULL    | NULL      | NULL       |
|          4 | Li       | Lily      | SCNU    | Guangzhou | NULL       |
|          5 | Chen     | Rain      | NULL    | Guangzhou | 1969-07-08 |
+------------+----------+-----------+---------+-----------+------------+
5 rows in set (0.00 sec)
```
使用 timestampdiff(YEAR,Birth,curdate())
```
MariaDB [personalFinancialDB]> SELECT LastName,FirstName,Birth,curdate(),timestampdiff(YEAR,Birth,curdate()) AS age FROM persons;   
+----------+-----------+------------+------------+------+
| LastName | FirstName | Birth      | curdate()  | age  |
+----------+-----------+------------+------------+------+
| Li       | Sharon    | 1993-02-26 | 2019-03-06 |   26 |
| Tang     | Jack      | NULL       | 2019-03-06 | NULL |
| Tang     | Ben       | NULL       | 2019-03-06 | NULL |
| Li       | Lily      | NULL       | 2019-03-06 | NULL |
| Chen     | Rain      | 1969-07-08 | 2019-03-06 |   49 |
+----------+-----------+------------+------------+------+
5 rows in set (0.04 sec)
```
使用 YEAR() 和 RIGHT()
```
# 未过生日 (RIGHT(curdate(),5)<RIGHT(Birth,5)) 为真即是1，所以年份相减结果-1

MariaDB [personalFinancialDB]> SELECT LastName,FirstName,Birth,curdate(), (year(curdate())-year(Birth))-(RIGHT(curdate(),5)<RIGHT(Birth,5)) AS age  from persons;
+----------+-----------+------------+------------+------+
| LastName | FirstName | Birth      | curdate()  | age  |
+----------+-----------+------------+------------+------+
| Li       | Sharon    | 1993-02-26 | 2019-03-06 |   26 |
| Tang     | Jack      | NULL       | 2019-03-06 | NULL |
| Tang     | Ben       | NULL       | 2019-03-06 | NULL |
| Li       | Lily      | NULL       | 2019-03-06 | NULL |
| Chen     | Rain      | 1969-07-08 | 2019-03-06 |   49 |
+----------+-----------+------------+------------+------+
5 rows in set (0.00 sec)
```
#### 找出某月过生日的人
```
MariaDB [personalFinancialDB]> select * from persons where month(Birth)='02';
+------------+----------+-----------+---------+------+------------+
| ID_persons | LastName | FirstName | Address | City | Birth      |
+------------+----------+-----------+---------+------+------------+
|          1 | Li       | Sharon    | NULL    | NULL | 1993-02-26 |
+------------+----------+-----------+---------+------+------------+
1 row in set (0.00 sec)
```
找下月出生的人可使用DATE_ADD(), DATE_ADD()允许在一个给定的日期上加上时间间隔。
```
MariaDB [(none)]> select date_add(curdate(),interval 1 month);
+--------------------------------------+
| date_add(curdate(),interval 1 month) |
+--------------------------------------+
| 2019-04-06                           |
+--------------------------------------+
1 row in set (0.04 sec)

```
当前月为12月，找下月出生的人，MOD(something,12)返回在0和11之间的一个数字，因此必须在MOD()后加1，否则我们将从11月(11)跳到1月(1)。
```
mysql> SELECT * FROM persons
    -> WHERE MONTH(Birth) = MOD(MONTH(CURDATE()), 12) + 1;
```

## 字符串函数
### 字符串长度
```
MariaDB [(none)]> select char_length('sharon');
+-----------------------+
| char_length('sharon') |
+-----------------------+
|                     6 |
+-----------------------+
1 row in set (0.00 sec)
```
### 连接两个字符串
无分隔符
```
MariaDB [(none)]> select concat('my','s','ql');
+-----------------------+
| concat('my','s','ql') |
+-----------------------+
| mysql                 |
+-----------------------+
1 row in set (0.02 sec)

MariaDB [(none)]> select concat('my',null,'ql');
+------------------------+
| concat('my',null,'ql') |
+------------------------+
| NULL                   |
+------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select concat(14.3);
+--------------+
| concat(14.3) |
+--------------+
| 14.3         |
+--------------+
1 row in set (0.01 sec)

MariaDB [(none)]> select concat(14.3,222);
+------------------+
| concat(14.3,222) |
+------------------+
| 14.3222          |
+------------------+
1 row in set (0.00 sec)
```
有分隔符
```
MariaDB [(none)]> select concat_ws(',','my','s','ql');
+------------------------------+
| concat_ws(',','my','s','ql') |
+------------------------------+
| my,s,ql                      |
+------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select concat_ws(',','my',null,'ql');
+-------------------------------+
| concat_ws(',','my',null,'ql') |
+-------------------------------+
| my,ql                         |
+-------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select concat_ws(',',1,1,1,3);
+------------------------+
| concat_ws(',',1,1,1,3) |
+------------------------+
| 1,1,1,3                |
+------------------------+
1 row in set (0.00 sec)
```
### 选值
```
MariaDB [(none)]> select elt(2,'a','b','c');
+--------------------+
| elt(2,'a','b','c') |
+--------------------+
| b                  |
+--------------------+
1 row in set (0.03 sec)

MariaDB [(none)]> select elt(3,'a','b','c');
+--------------------+
| elt(3,'a','b','c') |
+--------------------+
| c                  |
+--------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select elt(9,'a','b','c');
+--------------------+
| elt(9,'a','b','c') |
+--------------------+
| NULL               |
+--------------------+
1 row in set (0.00 sec)
```

```
MariaDB [(none)]> select export_set(5,'a','c',',',5);
+-----------------------------+
| export_set(5,'a','c',',',5) |
+-----------------------------+
| a,c,a,c,c                   |
+-----------------------------+
1 row in set (0.00 sec)

```
### 查找字符第一次出现位置
```
MariaDB [(none)]> select instr('dearxuany123123','123');
+--------------------------------+
| instr('dearxuany123123','123') |
+--------------------------------+
|                             10 |
+--------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select instr('dearxuany1','123');
+---------------------------+
| instr('dearxuany1','123') |
+---------------------------+
|                         0 |
+---------------------------+

```

### 比较查找
指定字符串是否存在于集合中
```
MariaDB [(none)]> select field('sharon','li','mei','sh');
+---------------------------------+
| field('sharon','li','mei','sh') |
+---------------------------------+
|                               0 |
+---------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select field('sharon','li','mei','sh','sharon');
+------------------------------------------+
| field('sharon','li','mei','sh','sharon') |
+------------------------------------------+
|                                        4 |
+------------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select field('sh','li','mei','sh','sharon');
+--------------------------------------+
| field('sh','li','mei','sh','sharon') |
+--------------------------------------+
|                                    3 |
+--------------------------------------+
1 row in set (0.01 sec)

MariaDB [(none)]> select find_in_set('sh','li,mei,sh,sharon');
+--------------------------------------+
| find_in_set('sh','li,mei,sh,sharon') |
+--------------------------------------+
|                                    3 |
+--------------------------------------+
1 row in set (0.00 sec)
```
### 替换
INSERT(str,pos,len,newstr) 
```
# 由第二位开始，替换4位字符
MariaDB [(none)]> select insert('sharon',2,4,'123');
+----------------------------+
| insert('sharon',2,4,'123') |
+----------------------------+
| s123n                      |
+----------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select insert('sharon',2,666,'123');
+------------------------------+
| insert('sharon',2,666,'123') |
+------------------------------+
| s123                         |
+------------------------------+
1 row in set (0.00 sec)

```
### 格式转换
FORMAT(X,D) 数字 X 的格式设置为 '#,###,###.##',以四舍五入的方式保留到小数点后 D 位, 返回结果为一个字符串。
```
MariaDB [(none)]> select format('1266666.55555',3);
+---------------------------+
| format('1266666.55555',3) |
+---------------------------+
| 1,266,666.556             |
+---------------------------+
1 row in set (0.00 sec)
```
### 大小写转换
```
MariaDB [(none)]> select lower('SHARON'),upper('acv');
+-----------------+--------------+
| lower('SHARON') | upper('acv') |
+-----------------+--------------+
| sharon          | ACV          |
+-----------------+--------------+
1 row in set (0.00 sec)
```
### 提取字符
left('str',n) 由左边开始数，前n个字符
```
MariaDB [(none)]> select left('sharon',3),right('sharon',2);
+------------------+-------------------+
| left('sharon',3) | right('sharon',2) |
+------------------+-------------------+
| sha              | on                |
+------------------+-------------------+
1 row in set (0.00 sec)
```
SUBSTRING(str,pos) , SUBSTRING(str FROM pos) SUBSTRING(str,pos,len) , SUBSTRING(str FROM pos FOR len)</br>
由字符串的第pos位开始提取后面len位字符，pos为负是由右边开始数
```
MariaDB [(none)]> select substring('sharon',-2);
+------------------------+
| substring('sharon',-2) |
+------------------------+
| on                     |
+------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select substring('sharon',2);
+-----------------------+
| substring('sharon',2) |
+-----------------------+
| haron                 |
+-----------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select substring('sharon',-2,1);
+--------------------------+
| substring('sharon',-2,1) |
+--------------------------+
| o                        |
+--------------------------+
1 row in set (0.01 sec)

```
### 按分隔符分割字符串
SUBSTRING_INDEX(str,delim,count)
```
MariaDB [(none)]> select substring_index('www.baidu.com','.',-2);
+-----------------------------------------+
| substring_index('www.baidu.com','.',-2) |
+-----------------------------------------+
| baidu.com                               |
+-----------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select substring_index('www.baidu.com','.',2);
+----------------------------------------+
| substring_index('www.baidu.com','.',2) |
+----------------------------------------+
| www.baidu                              |
+----------------------------------------+
1 row in set (0.00 sec)
```


## 其他内建函数
内建函数：</br>
SQL 拥有很多可用于计数和计算的内建函数。</br>
使用内建函数的语法：</br>
```
SELECT function(column_name)
FROM table_name
```
实例：
```
select avg(amount) from sakila.payment
```
### 合计函数 Aggregate functions
常用内建合计函数：AVG(column)、MAX(column)、MIN(column)、SUM(column)、COUNT(column)、 FIRST(column)、LAST(column)。</br>

#### GROUP BY
合计函数函数通常和 GROUP BY 合用，以便对不同种类的信息进行分组分析计算并获得想要的结果。
```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
```
实例：
在一个payment表中，计算各个客户各自在本店消费的总金额。
```
select customer_id, SUM(amount)
from sakila.payment
GROUP BY customer_id
```
进一步细分到各个客户在本店每日的消费总额。
```
select customer_id, payment_date,SUM(amount)
from sakila.payment
GROUP BY customer_id, payment_date
```
添加筛选，例如仅看客户2
```
select customer_id, payment_date,SUM(amount)
from sakila.payment
WHERE customer_id=2
GROUP BY customer_id, payment_date
```

#### HAVING
HAVING 用于和合计函数配合，设置条件进而对数据进行筛选。
类似于WHERE，但是WHERE语法上无法与合计函数一起用，所以就设计了HAVING这个东西。
```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```
实例：
在一个payment表中，计算各个客户各自在本店消费的总金额，筛选出消费总额大于100的用户。
```
select customer_id, SUM(amount)
from sakila.payment
GROUP BY customer_id
HAVING SUM(amount)>100
```
#### 最大最小值 MAX(column)、MIN(column)
注意：max和min的参数是column
```
MariaDB [personalFinancialDB]> select * from shop;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0001 | A      |  3.45 |
|    0001 | B      |  3.99 |
|    0002 | A      | 10.99 |
|    0003 | B      |  1.45 |
|    0003 | C      |  1.69 |
|    0003 | D      |  1.25 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
7 rows in set (0.00 sec)
```
价格最大的物品
```
MariaDB [personalFinancialDB]> select * from shop where price=max(price);
ERROR 1111 (HY000): Invalid use of group function
MariaDB [personalFinancialDB]> select * from shop where price=(select max(price)from shop);
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0004 | D      | 19.95 |
+---------+--------+-------+
1 row in set (0.06 sec)
```
各个物品最大的价格
```
MariaDB [personalFinancialDB]> select * from shop where price=(select max(price)from shop group by article);
ERROR 1242 (21000): Subquery returns more than 1 row
MariaDB [personalFinancialDB]> select max(price)from shop group by article;     
+------------+
| max(price) |
+------------+
|       3.99 |
|      10.99 |
|       1.69 |
|      19.95 |
+------------+
4 rows in set (0.00 sec)

MariaDB [personalFinancialDB]> select article,max(price)from shop group by article;
+---------+------------+
| article | max(price) |
+---------+------------+
|    0001 |       3.99 |
|    0002 |      10.99 |
|    0003 |       1.69 |
|    0004 |      19.95 |
+---------+------------+
4 rows in set (0.00 sec)
```
最大价格和最小价格对应的物品
```
MariaDB [personalFinancialDB]> select article,price from shop where price=(select max(price) from shop) or price=(select min(price) from shop);
+---------+-------+
| article | price |
+---------+-------+
|    0003 |  1.25 |
|    0004 | 19.95 |
+---------+-------+
2 rows in set (0.00 sec)
```
使用用户变量</br>
注意：用户变量只有本次登录有用，退出重登后要重新设置才能用
```
MariaDB [personalFinancialDB]> select @price_min:= min(price),@price_max:=max(price) from shop;
+-------------------------+------------------------+
| @price_min:= min(price) | @price_max:=max(price) |
+-------------------------+------------------------+
|                    1.25 |                  19.95 |
+-------------------------+------------------------+
1 row in set (0.00 sec)

MariaDB [personalFinancialDB]> select * from shop where price=@price_min or price=@price_max;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0003 | D      |  1.25 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
2 rows in set (0.00 sec)
```
### COALESCE(value,...) 
COALESCE(value,...) 返回 VALUE 中第一个非NULL值，全为null返回null
```
MariaDB [(none)]> select coalesce(null,2,null,6,7);
+---------------------------+
| coalesce(null,2,null,6,7) |
+---------------------------+
|                         2 |
+---------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> select coalesce(null,null);
+---------------------+
| coalesce(null,null) |
+---------------------+
|                NULL |
+---------------------+
1 row in set (0.01 sec)
```
### greatest() 找value的最大值最小值
注意greatest()的参数是values
```

MariaDB [(none)]> select max('a','b','c');
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''b','c')' at line 1
MariaDB [(none)]> select greatest('a','b','c');
+-----------------------+
| greatest('a','b','c') |
+-----------------------+
| c                     |
+-----------------------+
1 row in set (0.03 sec)

MariaDB [(none)]> select max(8,5,7);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '5,7)' at line 1
MariaDB [(none)]> select greatest(8,5,7);
+-----------------+
| greatest(8,5,7) |
+-----------------+
|               8 |
+-----------------+
1 row in set (0.00 sec)
```

### Scalar 函数
常用Scalar函数：</br>
* UCASE()  把字段的值转换为大写</br>
* LCASE()  把字段的值转换为小写</br>
* LEN()    返回文本字段中值的长度</br>
* MID()    从文本字段中提取字符</br>
```
SELECT MID(column_name,start[,length]) FROM table_name
```
参数：</br>
start  - 必需，规定开始位置（起始值是 1）</br>
length - 可选，要返回的字符数。如果省略，则 MID() 函数返回剩余文本</br>
* ROUND()  把数值字段舍入为指定的小数位数</br>
（未确定是四舍五入，还是向下圆整...）
```
SELECT ROUND(column_name,decimals) FROM table_name
```
参数：</br>
decimals - 必需，规定要返回的小数位数</br>
* FORMAT()  用于对字段的显示进行格式化</br>
普通字段调格式用FORMAT()，日期调格式用的DATE_FORMAT()</br>
```
SELECT FORMAT(column_name,format) FROM table_name
```
参数：</br>
format - 必需，规定格式</br>


# 拓展记录
## 关于删除一行数据时自增主键的处理
实例: 要求删除主键为3和4的重复数据
```
MariaDB [personalFinancialDB]> SELECT * FROM expenditure;
+----------------+---------------+---------------+-------------+------------+
| ID_expenditure | expenseDetail | expenseAmount | expenseDate | ID_persons |
+----------------+---------------+---------------+-------------+------------+
|              1 | Mucha Milktea |           2.7 | 2016-08-11  |          1 |
|              2 | Bubble        |          20.0 | 2016-08-11  |          1 |
|              3 | Mucha Milktea |           2.7 | 2016-08-11  |          1 |
|              4 | Bubble        |          20.0 | 2016-08-11  |          1 |
+----------------+---------------+---------------+-------------+------------+
```
用 delete 删除3和4的数据
```
DELETE FROM expenditure WHERE ID_expenditure=3;
DELETE FROM expenditure WHERE ID_expenditure=4;
```
重新插入新数据
```
MariaDB [personalFinancialDB]> SELECT * FROM expenditure;
+----------------+---------------+---------------+-------------+------------+
| ID_expenditure | expenseDetail | expenseAmount | expenseDate | ID_persons |
+----------------+---------------+---------------+-------------+------------+
|              1 | Mucha Milktea |           2.7 | 2016-08-11  |          1 |
|              2 | Bubble        |          20.0 | 2016-08-11  |          1 |
|              5 | Mucha Milktea |           2.7 | 2016-08-11  |          1 |
|              6 | Bubble        |          20.0 | 2016-08-11  |          1 |
+----------------+---------------+---------------+-------------+------------+
```
当主键设置了AUTO_INCREMENT，如果一行数据被删除，这行数据对应的主键号码是不会再次被分配的。</br>
例如，删除了主键号码为3、4的两行数据，之后再新增两行数据，这两行新数据的主键号码会被设置为5和6，在同一个表格中号码3和4不会再被使用了。</br>
一般情况下，主键的ID会根据实际情况设计，会使用MAX()检查当前所有ID的最大值，然后再将最大值加1后作为新ID插入到主键位置。</br>











