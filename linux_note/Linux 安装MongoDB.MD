# Linux上安装MongoDB
## 下载安装包
下载地址：https://www.mongodb.com/download-center#atlas</br>
最新版本为MongoDB 4.0，可是4.0貌似没有32位。</br>
支持32位的最高版本为3.2.20，下载地址：https://fastdl.mongodb.org/linux/mongodb-linux-i686-3.2.20.tgz</br>
执行以下命令，选择路径，下载安装包
```
cd /usr/local
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-i686-3.2.20.tgz

#输出结果
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 70.4M  100 70.4M    0     0   284k      0  0:04:13  0:04:13 --:--:--  340k
```

## 解压
```
tar -zxvf mongodb-linux-i686-3.2.20.tgz
```

## 查看安装路径
```
[root@centOSlearning mongodb-linux-i686-3.2.20]# pwd
/usr/local/mongodb-linux-i686-3.2.20
```

## 添加到PATH路径中
```
export PATH=/usr/local/mongodb-linux-i686-3.2.20/bin:$PATH
```

## 创建数据库目录
MongoDB的数据存储在data目录的db目录下，这个目录必须手动创建。</br>
注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。</br>
然后，创建一个日志文件，也记下它的路径。</br>

* 启动数据库目录路径
```
[root@centOSlearning data]# pwd
/usr/local/mongodb-linux-i686-3.2.20/data
```
* 日志文件路径
```
[sunnylinux@centOSlearning bin]$ pwd ./log
/usr/local/mongodb-linux-i686-3.2.20/bin
```

## 执行mongod命令来启动mongdb服务
这种方法不会开机自启动，也即是每次都要输入下面的命令才能打开。</br>
进入mongo安装目录中的bin目录</br>
```
cd /usr/local/mongodb-linux-i686-3.2.20/bin
[root@centOSlearning bin]# ./mongod --storageEngine=mmapv1 --journal
--dbpath=/usr/local/mongodb-linux-i686-3.2.20/data --logpath=/usr/local/mongodb-linux-i686-3.2.20/bin/log --logappend  --port=27017 --fork 

#看见这一行即是已经启动了
2018-07-20T18:19:12.059+0800 I NETWORK  [initandlisten] waiting for connections on port 27017
```

打开另外一个命令行界面，原来的那个不要关闭，打开MongoDB Shell</br>
进入mongo安装目录中的bin目录</br>
```
[sunnylinux@centOSlearning bin]$ ./mongo
MongoDB shell version: 3.2.20
connecting to: test
Welcome to the MongoDB shell.
```
由于它是一个JavaScript shell，可以运行一些简单算术运算:
```
> 2+2
4
> 3+4
7
```

## 设置为开机自动运行
将mongodb启动项目追加入rc.local保证mongodb在服务器开机时启动。
```
#现在暂且不想它这么做，所以暂且没有设置，仅记个方法
echo "/usr/local/mongodb-linux-i686-3.2.20/bin/mongod --storageEngine=mmapv1 --journal
--dbpath=/usr/local/mongodb-linux-i686-3.2.20/data --logpath=/usr/local/mongodb-linux-i686-3.2.20/bin/log --logappend  --port=27017 --auth" >> /etc/rc.local
```

## 关闭Mongodb
关闭Mongodb的方式有很多种，此处使用数据库命令。</br>
在Mongodb SHELL 里输入以下命令关闭 MongoDB 服务：
```
> use admin
switched to db admin
> db.shutdownServer()
#以下为输出
server should be down...
2018-07-21T01:51:03.601+0800 I NETWORK  [thread1] trying reconnect to 127.0.0.1:27017 (127.0.0.1) failed
2018-07-21T01:51:03.610+0800 W NETWORK  [thread1] Failed to connect to 127.0.0.1:27017, in(checking socket for error after poll), reason: errno:111 Connection refused
2018-07-21T01:51:03.610+0800 I NETWORK  [thread1] reconnect 127.0.0.1:27017 (127.0.0.1) failed failed 
> 
```
