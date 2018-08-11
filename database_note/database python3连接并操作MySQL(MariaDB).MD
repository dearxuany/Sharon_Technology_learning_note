# python3连接并操作MySQL(MariaDB)
Python3 使用 PyMySQL 连接数据库，Python2中则使用mysqldb。</br>
PyMySQL 遵循 Python 数据库 API v2.0 规范，并包含了 pure-Python MySQL 客户端库。</br>

详情参考：</br>
DB-API 2.0  https://www.python.org/dev/peps/pep-0249/ </br>
MySQL Reference Manuals: http://dev.mysql.com/doc/ </br>
MySQL client/server protocol: http://dev.mysql.com/doc/internals/en/client-server-protocol.html </br>
"Connector" channel in MySQL Community Slack: http://lefred.be/mysql-community-on-slack/ </br>
PyMySQL mailing list: https://groups.google.com/forum/#!forum/pymysql-users </br>


## PyMySQL 安装
参照：https://pypi.org/project/PyMySQL/ </br>
注意：需要root权限</br>

进入到python3所在文件夹
```
# cd /usr/local/python3
```
用pip3安装最新版的 PyMySQL
```
# pip3 install PyMySQL
You are using pip version 9.0.1, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
要求更新pip 版本到18.0，去更新一下，参照：
[Linux CentOS7 在python2、python3共存状态下安装pip2和pip3](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/linux_note/Linux%20CentOS7%20%E5%9C%A8python2%E3%80%81python3%E5%85%B1%E5%AD%98%E4%B8%8B%E5%AE%89%E8%A3%85pip2%E5%92%8Cpip3.MD)

重新运行PyMySQL安装命令
```
# pip3 install PyMySQL
raise ReadTimeoutError(self._pool, None, 'Read timed out.')
pip._vendor.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out.
```
出现Read timed out，可加大超时时间，将命令改为以下样式：
```
# pip3 --default-timeout=100 install -U PyMySQL
Successfully installed PyMySQL-0.9.2 asn1crypto-0.24.0 cffi-1.11.5 cryptography-2.3 idna-2.7 pycparser-2.18 six-1.11.0
```

## 数据库连接
### 连接前数据库准备
登录数据库，在mariaDB创建一个名为personalFinancialDB的数据库：</br>
```
MariaDB [(none)]> CREATE DATABASE personalFinancialDB ;   # linux上一定要用;号表示语句结束
Query OK, 1 row affected (0.09 sec)
MariaDB [(none)]> show databases;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| mysql               |
| performance_schema  |
| personalFinancialDB |
+---------------------+
4 rows in set (0.18 sec)
```
### python3连接数据库
#### 在已存在数据库中新建表
主要用到</br>
连接对象方法： db.cursor()、db.close()</br>
游标对象方法：cursor.execute(sql)</br>
</br>
编写脚本
```
#! /usr/bin/python3

import pymysql

# 打开数据库连接
db=pymysql.connect('localhost','username','password','personalFinancialDB')

# 使用cursor()方法创建一个游标对象cursor
cursor=db.cursor()

# SQL创建表
sql= """CREATE TABLE personalFinancialDB.persons(
        ID_persons int NOT NULL AUTO_INCREMENT,    -- ID自动增加，不能为空
        LastName varchar(255) NOT NULL,             -- 姓不能为空
        FirstName varchar(255),
        Address varchar(255),
        City varchar(255),
        PRIMARY KEY (ID_persons)                   -- ID_persons为主键
        )"""

# 执行SQL语句
cursor.execute(sql)

# 关闭数据库
db.close()
```           
执行脚本后，登录数据库看看是否正常创建表
```
MariaDB [(none)]> use personalFinancialDB
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [personalFinancialDB]> SHOW TABLES ;
+-------------------------------+
| Tables_in_personalFinancialDB |
+-------------------------------+
| persons                       |
+-------------------------------+
1 row in set (0.00 sec)
```

#### 在已存在的表中插入数据
主要用到</br>
连接对象方法：db.cursor()、 db.commit() 、db.rollback()、db.close()</br>
游标对象方法：cursor.execute(sql)</br>
删除、更新操作和插入操作的写法一样，只是SQL语句的内容不一样。</br>
</br>
编写脚本
```
#! /usr/bin/python3

import pymysql

db=pymysql.connect('localhost','username','password','personalFinancialDB')

cursor=db.cursor()

# sql语句进行插入操作
sql= """INSERT INTO personalFinancialDB.persons (LastName,FirstName)
        VALUES ('Li','Sharon');"""

# 尝试插入数据库，如果不成功则回滚
try:
    cursor.execute(sql)
    #提交到数据库执行
    db.commit()
except:
    db.rollback()

db.close()
```
数据库内查询一下
```
MariaDB [personalFinancialDB]> SELECT * FROM persons;
+------------+----------+-----------+---------+------+
| ID_persons | LastName | FirstName | Address | City |
+------------+----------+-----------+---------+------+
|          1 | Li       | Sharon    | NULL    | NULL |
+------------+----------+-----------+---------+------+
1 row in set (0.00 sec)
```
#### 数据库查询操作
主要用到</br>
连接对象方法：db.cursor()、db.close()</br>
游标对象方法：cursor.execute(sql)、cursor.fetchone()、cursor.fetchall()</br>
Python查询Mysql使用 fetchone() 方法获取单条数据, 使用fetchall() 方法获取多条数据，返回值的类型是元组。</br>
</br>
编写脚本
```
#! /user/bin/python3

import pymysql

db=pymysql.connect('localhost','sharonli','sharonli','personalFinancialDB')

cursor=db.cursor()

sql1="SELECT * FROM persons WHERE ID_persons=1;"
sql2="SELECT * FROM expenditure;"

try:
    cursor.execute(sql1)
    #使用fetchone()方法返回单行数据
    result=cursor.fetchone()
    print(result)
except:
    print ("Error: unable to fetch data")

try:
    cursor.execute(sql2)
    #使用方法fetchall()返回多行数据
    results=cursor.fetchall()
    print(results)
except:
    print ("Error: unable to fetch data")

db.close()
```
执行结果
```
(1, 'Li', 'Sharon', None, None)
(
(1, 'Mucha Milktea', Decimal('2.7'), datetime.date(2018, 8, 11), 1), 
(2, 'Bubble', Decimal('20.0'), datetime.date(2018, 8, 11), 1), 
(7, 'Cake', Decimal('32.0'), datetime.date(2018, 8, 11), 2), 
(8, 'Coffee', Decimal('20.0'), datetime.date(2018, 8, 11), 3)
)
```
注：返回值是没有格式的，方便看结构才改的换行。</br>
上面两个方法的返回结果都是元组，之后的处理就是python上的迭代问题了。</br>
还有就是，数据库中的NULL会被Python转换为None。</br>
