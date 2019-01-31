# Linux yum安装 php、php-fpm、php-mysql 
FastCGI 进程管理器（FPM） http://php.net/manual/zh/install.fpm.php
```
# 安装php
sudo yum install php
# 安装php-fpm
sudo yum install php-fpm
# 安装php-mysql
sudo yum install php-mysql
```
查看版本
```
$ php -v
PHP 5.4.16 (cli) (built: Oct 30 2018 19:42:25)
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies

$ php-fpm -v
PHP 5.4.16 (fpm-fcgi) (built: Oct 30 2018 19:44:03)
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies
```
因为是用yum安装，所以它们的配置文件会在/etc下面
```
[sunnylinux@centOSlearning etc]$ ls|grep php
php.d
php-fpm.conf  # FPM配置文件
php-fpm.d
php.ini # php配置文件
```
常用命令</br>
The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
```
service php-fpm start
service php-fpm status
service php-fpm restart
service php-fpm stop
service php-fpm reload
```

