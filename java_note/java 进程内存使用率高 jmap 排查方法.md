# jmap 排查 java 进程内存使用率高
## 步骤
glances 找出服务器中 CPU 占用率高的进程
```
glances
```
临时修改程序用户的 shell 为可登录用户并切换到该用户<br/>
注意：使用 jmap 必须在 java 进程启动用户下，否则会报错<br/>
```
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
usermod -s /bin/bash tomcat
su - tomcat
```
查看 java 进程号
```
ps -ef|grep tomcat
```
使用进程号生成 javadump 文件<br/>
注意：<br/>
javadump 会随程序规模及运行时间增大而增大，一般生产在几十G左右；<br/>
生成 javadump 文件会导致程序短暂挂起，容易影响生产服务稳定，不能轻易使用。<br/>
```
jmap -dump:format=b,file=/tmp/tomcat-javadump-20200102 pid
```
从服务器下载 javadump 文件后可使用 jdk 自带的 jvisualvm 工具进行分析<br/>
注意：javadump 文件太大可能导致个人电脑无法打开，但 javadump 文件为二进制文件也不宜切割 <br/>
