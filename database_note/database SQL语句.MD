# SQL语句
SQL 对大小写不敏感，SELECT 与 select 是相同的，新建的数据库时名称要用小写，语句用大写会比较好分辨。
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
SELECT * FROM database_name.table_name WHERE (column_name1 >= value AND column_name2 = value) OR column_name3 <= value;
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
INSERT INTO database_name.table_name (column1,column3) VALUES (value1,value3)
```
(注意插入是没有输出的！)
```
insert into university.department values ('math','harry',3000)
insert into university.department (building,budget) values ('iron',6000)  #这条好像没成功
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
