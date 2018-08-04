# SQL基础语句
SQL特性： 
* 对大小写不敏感，SELECT 与 select 是相同的，新建的数据库时名称要用小写，语句用大写会比较好分辨;
* 对空格不敏感，可以换行，增加可读性；
* 多条SQL语句必须用分号；分隔，单条语句部分RDBMS不需要用分号分隔;
* 不同数据库中的SQL语法会有略微不同（此处主要记录mySQL）。
```
常用语句
SELECT - 从数据库中提取数据
UPDATE - 更新数据库中的数据
DELETE - 从数据库中删除数据
INSERT INTO - 向数据库中插入新数据
CREATE DATABASE - 创建新数据库
ALTER DATABASE - 修改数据库
CREATE TABLE - 创建新表
ALTER TABLE - 变更（改变）数据库表
DROP TABLE - 删除表
CREATE INDEX - 创建索引（搜索键）
DROP INDEX - 删除索引
```
```
注释
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
values ('iron',6000);  #这条好像没成功
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

# DROP、TRUNCATE、 DELETE 的区别
truncate table 命令将快速删除数据表中的所有记录，但保留数据表结构。这种快速删除与 delete from 数据表的删除全部数据表记录不一样，delete 命令删除的数据将存储在系统回滚段中，需要的时候，数据可以回滚恢复，而 truncate 命令删除的数据是不可以恢复的。
## 相同点
truncate 和不带 where 子句的 delete, 以及 drop 都会删除表内的数据。
## 不同点 
* truncate 和 delete 只删除数据不删除表的结构(定义) ，drop 语句将删除表的结构被依赖的约束(constrain), 触发器(trigger), 索引(index); 依赖于该表的存储过程/函数将保留, 但是变为 invalid 状态。</br>
* delete 语句是 dml, 这个操作会放到 rollback segement 中, 事务提交之后才生效; 如果有相应的 trigger, 执行的时候将被触发。 truncate, drop 是 ddl, 操作立即生效, 原数据不放到 rollback segment 中, 不能回滚。 操作不触发 trigger。</br>
* delete 语句不影响表所占用的 extent, 高水线(high watermark)保持原位置不动。 显然 drop 语句将表所占用的空间全部释放 。 truncate 语句缺省情况下见空间释放到 minextents 个 extent, 除非使用 reuse storage; truncate会将高水线复位(回到最开始)。</br>
* 速度：一般来说: drop > truncate > delete 。</br>
* 安全性: 小心使用 drop 和 truncate, 尤其没有备份的时候。</br>
使用上, 想删除部分数据行用 delete, 注意带上 where 子句。 回滚段要足够大。</br>
想删除表, 当然用 drop。</br>
想保留表而将所有数据删除。如果和事务无关, 用 truncate 即可。 如果和事务有关, 或者想触发 trigger, 还是用 delete。</br>
如果是整理表内部的碎片, 可以用 truncate 跟上 reuse stroage, 再重新导入/插入数据。</br>

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
ID int NOT NULL AUTO_INCREMENT,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (ID)
)
```
尝试插入数据
```
insert into personalFinancialDB.persons (LastName,FirstName)
VALUES ('Li','Sharon')
```
ID自动增加为1，插入上面的values，其他位置为null。

### 数据类型 data type
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

* 容纳可变长度的字符串（可容纳字母、数字以及特殊的字符），在括号中规定字符串的最大长度；</br>
varchar(size)</br>

* 容纳日期：</br>
date(yyyymmdd)</br>


### 约束 Constraints
约束用于限制加入表的数据的类型；</br>
可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。</br>
</br>
主要有以下几种约束：

* NOT NULL</br>
强制列不接受 NULL 值，如果不向字段添加值，就无法插入新记录或者更新记录。

* UNIQUE</br>
唯一标识数据库表中的每条记录，为列或列集合提供了唯一性的保证，每个表可以有多个 UNIQUE 约束。

* PRIMARY KEY</br>
主键，唯一标识数据库表中的每条记录，不能包含 NULL 值，每个表都应该有一个主键，并且每个表只能有一个主键。</br>
使用多列作为主键，可使用以下语法：
```
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```

* FOREIGN KEY</br>
外键，一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。</br>
用于预防破坏表之间连接的动作，能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。</br>

* CHECK</br>
CHECK 用于限制列中的值的范围。</br>
如果对单个列定义 CHECK 约束，那么该列只允许特定的值。</br>
如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。</br>

* DEFAULT
插入默认值，如果没有规定其他的值，那么会将默认值添加到所有的新记录。</br>























