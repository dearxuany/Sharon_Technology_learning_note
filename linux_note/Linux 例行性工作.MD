# Linux 例行性工作 crontab

## linux 工作调度方式
linux下有两种工作调度方式：</br>
* at 命令 </br>
处理仅执行一次就结束的调度命令，需有atd服务支持，CentOS默认启动atd服务。</br>
* crontab 命令 </br>
按一定周期循环执行调度命令，由crond服务支持。除使用crontab命令来设置任务外，还可以通过修改/etc/crontab设置调度任务。</br>

linux上常见例行性工作：</br>
log rotate - 定期将旧日志文件挪到合适的地方，腾出空间；</br>
log watch - 日志分析；</br>
locate数据库的 updatedb ；</br>
man page数据库的建立；</br>
RPM 软件日志文件新建；</br>
删除临时文件：调度tmpwatch命令来删除临时文件；</br>
与网络服务有关的分析行为。</br>


## 循环执行的例行性工作调度 crontab
### cron 服务
由于cron 是Linux的service（deamon），可以用以下的方法启动、关闭这个服务： </br>
```
/sbin/service crond start //启动服务 
/sbin/service crond stop //关闭服务 
/sbin/service crond restart //重启服务 
/sbin/service crond reload //重新载入配置 
```
cron开机自启设置 </br>
在/etc/rc.d/rc.local这个脚本的末尾加上： </br>
```
/sbin/service crond start 
```
因为linux上很多服务本身就是用cron来控制的，所以linux上cron服务是默认开启的。</br>
</br>
查看服务状态
```
# service crond status
```
#### 用户设置
由于安全性问题，可限制使用crontab的用户账户。可将允许或拒绝的用户写入以下两个文件的其中一个，allow优先级大于deny，系统默认保留deny。为防止混乱最好只保留一个文件且一个账号一行。任何使用者只要不被列入 /etc/cron.deny 当中，那么他就可以直接下达crontab -e 去编辑自己的例行性命令。
```
/etc/cron.allow
/etc/cron.deny
```
用户使用crontab新建调度任务后，此任务会被记录到/var/spool/cron/，用账号来辨别，此文件禁止用vim编辑。

#### crontab语法
```
# crontab [-u username] [-l|-e|-r]
选项与参数：
-u  ：只有 root 才能进行这个任务，亦即帮其他使用者建立/移除 crontab 工作排程；
-e  ：编辑 crontab 的工作内容
-l  ：查阅 crontab 的工作内容
-r  ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑。
```
查询crontab 工作内容
```
# crontab -u root -l 
no crontab for root
# crontab -u sunnylinux -l
no crontab for sunnylinux
```
查看cron 运行记录
```
# tail -f /var/log/cron
Aug 31 20:00:01 centOSlearning crond[1210]: (/usr/sbin/ntpdate) ERROR (getpwnam() failed)

# tail -f /var/log/cron
Sep  1 04:00:01 centOSlearning CROND[5251]: (root) CMD (ntpdate time.windows.com cn.pool.ntp.org)
Sep  2 16:20:02 centOSlearning CROND[5289]: (root) CMD (/usr/lib/sa/sa1 1 1)
Sep  2 16:25:59 centOSlearning crontab[5318]: (root) LIST (root)
```

#### cron相关的系统配置文件 /etc/crontab
一般会直接编辑文件 /etc/crontab 来设置工作调度，而不是用crontab -e 来设置，编辑这个文件必须有root权限。</br>
cron服务最低检测限制是“分钟”，所以cron会每分钟去读一次/etc/crontab和/var/spool/cron/，所以在/etc/crontab编辑完工作调度任务后，cron就会自动执行他们，但最好还是重新启动一下crond服务。
```
# vim /etc/crontab
SHELL=/bin/bash    # 何种shell接口
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root       # 若有STOUT，以e-mail推送数据给谁

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
0 */2 * * * root ntpdate time.windows.com cn.pool.ntp.org
```
* MAILTO=root </br>
例行性工作出错或有STOUT时，以邮件形式发送数据的收件人。默认为root，可以直接填邮箱地址。如果一直出错，而又不想接受邮件，可以设置为/dev/null，将输出放到垃圾桶当中</br>
* user-name</br>
使用/etc/crontab 来编辑例行性工作必须填写user-name 即后面执行后续命令的对象。</br>
* job definition 格式</br>
时间格式：</br>
除了文件内显示的数字外，还有几个个特殊的符号可用，分别为星号、斜杠、减号、逗号。</br>
```
* 代表所有的取值范围内的数字；</br>
/ 代表每的意思，"*/5"表示每5个单位；</br>
- 代表从某个数字到某个数字，即某个区间；</br>
, 分开几个离散的数字，可用于分散任务执行时间，以免多个任务在同一时间执行。</br>
```
```
# 每天早上6点 
0 6 * * * echo "Good morning." >> /tmp/test.txt 
# 每两个小时 
0 */2 * * * echo "Have a break now." >> /tmp/test.txt 
# 晚上11点到早上8点之间每两个小时，早上八点 
0 23-7/2，8 * * * echo "Have a good dream：）" >> /tmp/test.txt 
# 每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点
0 11 4 * 1-3 command line 
# 1月1日早上4点
0 4 1 1 * command line 
```
注意：</br>
单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。</br>
循环周期，周和日月不可同时并存，仅能单独用周或者日、月来作为循环周期，否则会出错。</br>


命令格式：</br>
job definition 的命令格式有两种写法</br>
1.直接写要执行的命令，即像上面ntpdate那样写；</br>
2.先创建一个目录（根据执行周期创建），然后将要执行的工作调度脚本放在这些目录下。如，让系统每小时主动执行某个命令，可将命令写成脚本，然后放在cron.hourly下面，也可以自己用run-parts 创建拥有不同循环周期的目录。</br>
</br>
以下为系统已经新建好的目录
```
# cd /etc
# ls -al
drwxr-xr-x.   2 root root       54 5月  12 07:50 cron.d
drwxr-xr-x.   2 root root       57 5月  12 07:50 cron.daily
-rw-------.   1 root root        0 4月  11 09:49 cron.deny
drwxr-xr-x.   2 root root       41 5月  12 07:50 cron.hourly
drwxr-xr-x.   2 root root        6 6月  10 2014 cron.monthly
-rw-r--r--.   1 root root      509 9月   2 16:50 crontab
drwxr-xr-x.   2 root root        6 6月  10 2014 cron.weekly
```
目录规划
```
# 新建一个目录，将脚本放置于此目录下
mkdir /root/runcron
# 修改/etc/corntab给目录规划一个循环周期，如每两分钟执行目录下所有文件
*/2 * * * * root run-parts /root/runcron
```

#### anacron
anacron 是cron服务下的一个程序。</br>
当linux关机时，一些定时的crontab任务则无法正常执行。 anacron 在linux系统开机的时候或在crontab中指定的时候运行，读取timestamps分析检测关机时没有被运行的crontab任务，随后anacron会运行这些错误的任务，运行完成后anacron会自动停止。 </br>
anacron 会以一天、七天、一个月为周期检测系统未进行的crontab任务。anacron 在centOS中已被加入到crontab调度。anacron并不需要额外设置，使用默认值即可。</br>
```
# ll /etc/cron*/*ana*
-rwxr-xr-x. 1 root root 392 4月  11 09:49 /etc/cron.hourly/0anacron
```
这里看到，anacron是每小时进行一次的，为保证anacron优先运行，所以文件名前面有个0。
