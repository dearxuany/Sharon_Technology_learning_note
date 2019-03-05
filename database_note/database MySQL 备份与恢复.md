# database MySQL 备份与恢复
## 备份
备份与导出的区别：</br>
* 导出的文件只是保存数据库中的数据；</br>
* 而备份，则是把数据库的结构，包括数据、约束、索引、视图等全部另存为一个文件。</br>

### mysqldump 备份
mysqldump 是 MySQL 用于备份数据库的实用程序。</br>
它主要产生一个 SQL 脚本文件，其中包含从头重新创建数据库所必需的命令 CREATE TABLE INSERT 等。</br>
mysqldump 是一个备份工具，因此该命令是在终端中执行的，而不是在 mysql 交互环境下。</br>

```
MariaDB [(none)]> show databases
    -> ;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| falcon              |
| mysql               |
| performance_schema  |
| personalFinancialDB |
+---------------------+
5 rows in set (0.00 sec)
```

#### 备份整个数据库
```
$ mysqldump -u sharonli -p  personalFinancialDB>pfbackup.sql
Enter password:

$ ls|grep sql
pfbackup.sql

$ cat pfbackup.sql
-- MySQL dump 10.14  Distrib 5.5.56-MariaDB, for Linux (i686)
--
-- Host: localhost    Database: personalFinancialDB
-- ------------------------------------------------------
-- Server version       5.5.56-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `expenditure`
--

DROP TABLE IF EXISTS `expenditure`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `expenditure` (
  `ID_expenditure` int(11) NOT NULL AUTO_INCREMENT,
  `expenseDetail` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `expenseAmount` decimal(7,1) DEFAULT NULL,
  `expenseDate` date DEFAULT NULL,
  `ID_persons` int(11) DEFAULT NULL,
  PRIMARY KEY (`ID_expenditure`),
  KEY `ID_persons` (`ID_persons`),
  CONSTRAINT `expenditure_ibfk_1` FOREIGN KEY (`ID_persons`) REFERENCES `persons` (`ID_persons`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `expenditure`
--

LOCK TABLES `expenditure` WRITE;
/*!40000 ALTER TABLE `expenditure` DISABLE KEYS */;
INSERT INTO `expenditure` VALUES (1,'Mucha Milktea',2.7,'2018-08-11',1),(2,'Bubble',20.0,'2018-08-11',1),(7,'Cake',32.0,'2018-08-11',2),(8,'Coffee',20.0,'2018-08-11',3);
/*!40000 ALTER TABLE `expenditure` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `persons`
--

DROP TABLE IF EXISTS `persons`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `persons` (
  `ID_persons` int(11) NOT NULL AUTO_INCREMENT,
  `LastName` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `FirstName` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `Address` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `City` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`ID_persons`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `persons`
--

LOCK TABLES `persons` WRITE;
/*!40000 ALTER TABLE `persons` DISABLE KEYS */;
INSERT INTO `persons` VALUES (1,'Li','Sharon',NULL,NULL),(2,'Tang','Jack',NULL,NULL),(3,'Tang','Ben',NULL,NULL);
/*!40000 ALTER TABLE `persons` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2019-03-06  0:26:54

```
#### 备份单个表
```
$ mysqldump -u sharonli -p  personalFinancialDB expenditure>pfbackup_exp.sql
Enter password:
```
```
$ cat pfbackup_exp.sql
-- MySQL dump 10.14  Distrib 5.5.56-MariaDB, for Linux (i686)
--
-- Host: localhost    Database: personalFinancialDB
-- ------------------------------------------------------
-- Server version       5.5.56-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `expenditure`
--

DROP TABLE IF EXISTS `expenditure`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `expenditure` (
  `ID_expenditure` int(11) NOT NULL AUTO_INCREMENT,
  `expenseDetail` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `expenseAmount` decimal(7,1) DEFAULT NULL,
  `expenseDate` date DEFAULT NULL,
  `ID_persons` int(11) DEFAULT NULL,
  PRIMARY KEY (`ID_expenditure`),
  KEY `ID_persons` (`ID_persons`),
  CONSTRAINT `expenditure_ibfk_1` FOREIGN KEY (`ID_persons`) REFERENCES `persons` (`ID_persons`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `expenditure`
--

LOCK TABLES `expenditure` WRITE;
/*!40000 ALTER TABLE `expenditure` DISABLE KEYS */;
INSERT INTO `expenditure` VALUES (1,'Mucha Milktea',2.7,'2018-08-11',1),(2,'Bubble',20.0,'2018-08-11',1),(7,'Cake',32.0,'2018-08-11',2),(8,'Coffee',20.0,'2018-08-11',3);
/*!40000 ALTER TABLE `expenditure` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2019-03-06  0:54:16

```
注意：备份完注意检查备份文件是否为空！！！
## 恢复
恢复数据库可以使用source来恢复，也可以使用 mysql 语句来恢复。
### 使用 mysql 语句来恢复
#### 恢复整个数据库
先登录mysql然后新建一个空数据库
```
$ mysql -u sharonli -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 13
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE backuptest;
Query OK, 1 row affected (0.03 sec)

MariaDB [(none)]> SHOW DATABASES;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| backuptest          |
| falcon              |
| mysql               |
| performance_schema  |
| personalFinancialDB |
+---------------------+
6 rows in set (0.00 sec)

MariaDB [(none)]> use backuptest
Database changed
MariaDB [backuptest]> show tables;
Empty set (0.00 sec)
```
将之前的备份恢复到这个新建的数据库当中
```
$ mysql -u sharonli -p backuptest<pfbackup.sql
Enter password:
```
```
$ mysql -u sharonli -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 29
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use backuptest
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [backuptest]> show tables;
+----------------------+
| Tables_in_backuptest |
+----------------------+
| expenditure          |
| persons              |
+----------------------+
2 rows in set (0.00 sec)

```

