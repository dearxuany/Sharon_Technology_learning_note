# Linux 安装配置MySQL（MariaDB）
前言：</br>
本来打算安装MySQL的，可是途中发现一些必须的32位i686的镜像无法找到，然后CentOS7推荐安装MariaDB，所以就变成安装MariaDB了。

## 关于MariaDB
MariaDB 数据库管理系统是 MySQL 的一个分支，主要由开源社区在维护，采用 GPL 授权许可。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

## 安装mariaDB服务端和客户端
需要root权限
```
yum install mariadb-server mariadb 
yum install mysql
```
## 查看版本
有版本显示即是安装成功了
```
mysqladmin --version
# 输出
mysqladmin  Ver 9.0 Distrib 5.5.56-MariaDB, for Linux on i686
```
## mariadb数据库的相关命令
```
systemctl start mariadb 	#启动MariaDB
systemctl stop mariadb		#停止MariaDB
systemctl restart mariadb 	#重启MariaDB
systemctl status mariadb	#查看状态
systemctl enable mariadb	#设置开机启动
```

### 开机启动输出
```
# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
```
### 登录到数据库
登录到mariadb，此时root账户的密码为空。
```
# mysql -uroot
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

### 相关安全配置
启动数据库后，对MariaDB进行相关简单配置
```
# mysql_secure_installation
```
第一次设置，root的密码是空白的，会提示是否重新设置。
可以不设，这样谁都可以登录使用mariaDB而且安装也会更加顺利，但是如果是生产环境，需要记住移除这样的选择。
```
Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
```
要求输入两次新密码
```
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!
```
其他配置：
是否删除匿名用户、是否允许远程登录、 是否删除test数据库、是否重新加载权限表。
按回车，默认“是”。
```
Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
更改密码后的登录命令，会提示输入密码
```
# mysql -u root -p
Enter password:
```

## 配置字符集（按需求设置）
详情参照官网：</br>
https://mariadb.com/kb/zh-cn/setting-character-sets-and-collations/
### 查看字符设置
登录到mariadb账户，查看字符设置
```
MariaDB [(none)]> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.10 sec)
```
```
MariaDB [(none)]> show variables like "%collation%";
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
3 rows in set (0.01 sec)
```
看到上面部分位置标的是 latin1，要把他们改为utf8。

### 更改字符配置
查看文件/etc/my.cnf，看文件中是否含有 !includedir /etc/my.cnf.d 这一句，该句说明在该配置文件中引入/etc/my.cnf.d 目录下的配置文件。
```
# cat /etc/my.cnf
```
#### 修改server.cnf
去到/etc/my.cnf.d/ 目录下，找到server.cnf文件，用vim打开
```
# cd /etc/my.cnf.d/
# vim server.cnf
```
在[mysqld]标签下添加以下内容：
```
init_connect='SET collation_connection = utf8_unicode_ci' 
init_connect='SET NAMES utf8' 
character-set-server=utf8 
collation-server=utf8_unicode_ci 
skip-character-set-client-handshake
```
#### 修改client.cnf
```
# vim client.cnf
```
在[client]标签下添加： 
```
default-character-set=utf8
```
#### 修改 mysql-clients.cnf
```
# vim mysql-clients.cnf
```
在[mysql]标签下添加：
```
default-character-set=utf8
```
#### 重启数据库并查看字符配置
```
# systemctl restart mariadb
# mysql -u root -p
Enter password: 
```
```
MariaDB [(none)]> show variables like "%character%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```
```
MariaDB [(none)]> show variables like "%collation%";
+----------------------+-----------------+
| Variable_name        | Value           |
+----------------------+-----------------+
| collation_connection | utf8_unicode_ci |
| collation_database   | utf8_unicode_ci |
| collation_server     | utf8_unicode_ci |
+----------------------+-----------------+
3 rows in set (0.00 sec)
```

## 添加新用户并设置权限
### 创建用户
```
create user username@localhost identified by 'password';
```
创建用户有很多种不同方式，本次使用以下这种方式：
创建了一个名为username的用户，并允许其可从任意一台主机来连接服务器。（localhost由%代替）。
这种方式最为方便，但对系统最不安全。

### 授权外网登录
```
grant all privileges on *.* to username@'%' identified by 'password';
```

### 查看用户
```
MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [mysql]> select host,user from user;
MariaDB [mysql]> select  host,user,password from user;
```

### 连接数据库
语法：
```
$ mysql -h [localhost | IP] -P [port] -u [username] -p [password]
```
例如：连接本机 ，端口为3306，用户名为root，密码为：123456，命令为：
```
$ mysql -u root -p
```
执行后会提示输入密码，此时输入正确密码，成功登陆会出现MariaDB命令行提示符。 
在MariaDB服务器登录时可省略-h参数，端口为默认3306端口时可省略-P参数。
```
$ mysql -u [username] -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
