# jstack 排查 java 进程 CPU 占用率高
## 步骤
临时修改程序用户 jetty 的 shell 为可登录用户并切换到jetty用户
注意：使用 jstack 必须在 java 进程启动用户下，否则会报错
```
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow
usermod -s /bin/bash jetty
su - jetty
```

查看jetty进程，找到进程号
```
ps -ef|grep jetty
jetty     2416  2386 99 Sep20 ?        6-10:16:07 /sdata/usr/local/jdk/jre/bin/java -Xms5120m -Xmx5120m -Xmn1800m -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/sdata/var/log/jetty/gc.log -javaagent:/sdata/usr/local/pinpoint-agent/pinpoint-bootstrap-1.8.4.jar -Djava.io.tmpdir=/tmp -Djetty.home=/sdata/usr/local/jetty -Djetty.base=/sdata/usr/local/jetty -Dpinpoint.agentId=****-04 -Dpinpoint.applicationName=prd_**** -Djetty.logging.dir=/sdata/var/log/jetty/logs -cp /sdata/usr/local/jetty/lib/apache-jsp/org.eclipse.jdt.core.compiler.ecj-4.4.2.jar:/sdata/usr/local/jetty/lib/apache-jsp/org.eclipse.jetty.apache-jsp-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/apache-jsp/org.mortbay.jasper.apache-el-8.0.33.jar:/sdata/usr/local/jetty/lib/apache-jsp/org.mortbay.jasper.apache-jsp-8.0.33.jar:/sdata/usr/local/jetty/lib/apache-jstl/org.apache.taglibs.taglibs-standard-impl-1.2.5.jar:/sdata/usr/local/jetty/lib/apache-jstl/org.apache.taglibs.taglibs-standard-spec-1.2.5.jar:/sdata/usr/local/jetty/resources:/sdata/usr/local/jetty/lib/servlet-api-3.1.jar:/sdata/usr/local/jetty/lib/jetty-schemas-3.1.jar:/sdata/usr/local/jetty/lib/jetty-http-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-server-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-xml-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-util-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-io-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-jmx-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-jndi-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jndi/javax.mail.glassfish-1.4.1.v201005082020.jar:/sdata/usr/local/jetty/lib/jndi/javax.transaction-api-1.2.jar:/sdata/usr/local/jetty/lib/jetty-security-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-servlet-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-webapp-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-deploy-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-plus-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/jetty-annotations-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/annotations/asm-6.0.jar:/sdata/usr/local/jetty/lib/annotations/asm-commons-6.0.jar:/sdata/usr/local/jetty/lib/annotations/javax.annotation-api-1.2.jar:/sdata/usr/local/jetty/lib/websocket/javax.websocket-api-1.0.jar:/sdata/usr/local/jetty/lib/websocket/javax-websocket-client-impl-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/javax-websocket-server-impl-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/websocket-api-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/websocket-client-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/websocket-common-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/websocket-server-9.3.27.v20190418.jar:/sdata/usr/local/jetty/lib/websocket/websocket-servlet-9.3.27.v20190418.jar org.eclipse.jetty.xml.XmlConfiguration /tmp/start_8868297810396710550.properties /sdata/usr/local/jetty/etc/home-base-warning.xml /sdata/usr/local/jetty/etc/jetty.xml /sdata/usr/local/jetty/etc/jetty-http.xml /sdata/usr/local/jetty/etc/jetty-jmx.xml /sdata/usr/local/jetty/etc/jetty-deploy.xml /sdata/usr/local/jetty/etc/jetty-plus.xml /sdata/usr/local/jetty/etc/jetty-annotations.xml /sdata/usr/local/jetty/etc/jetty-logging.xml /sdata/usr/local/jetty/etc/jetty-started.xml
```

根据进程号找到使用较高的线程号
```
top -Hp 2416
 
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                      
 2680 jetty     20   0 9814.6m   6.0g 139820 R 87.2 38.8   2682:06 java                                                                                                                                         
 2243 jetty     20   0 9814.6m   6.0g 139820 S  0.5 38.8   7:22.13 java                                                                                                                                         
 2272 jetty     20   0 9814.6m   6.0g 139820 S  0.5 38.8   5:55.80 java                                                                                                                                         
20890 jetty     20   0 9814.6m   6.0g 139820 R  0.5 38.8   3:59.88 java
```

根据找到的线程号来找对应的十六进制代码
```
-bash-4.2$ printf "%x\n" 3301
ce5
```

使用jstack根据进程号查找线程信息，并将信息输出到文件中，使用对应的十六进制代码来找到对应的问题
```
jstack 2416
 
 
"Thread-29" #195 prio=5 os_prio=0 tid=0x00007f9495110000 nid=0xce5 runnable [0x00007f93ec5c6000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.Throwable.fillInStackTrace(Native Method)
        at java.lang.Throwable.fillInStackTrace(Throwable.java:783)
        - locked <0x000000079c227e58> (a redis.clients.jedis.exceptions.JedisConnectionException)
        at java.lang.Throwable.<init>(Throwable.java:287)
        at java.lang.Exception.<init>(Exception.java:84)
        at java.lang.RuntimeException.<init>(RuntimeException.java:80)
        at redis.clients.jedis.exceptions.JedisException.<init>(JedisException.java:15)
        at redis.clients.jedis.exceptions.JedisConnectionException.<init>(JedisConnectionException.java:15)
        at redis.clients.util.Pool.getResource(Pool.java:50)
        at redis.clients.jedis.JedisPool.getResource(JedisPool.java:99)
        at com.jfinal.plugin.redis.Cache.getJedis(Cache.java:1253)
        at com.woniubaoxian.common.utils.BnailRedisUtil.llen(BnailRedisUtil.java:85)
        at com.woniubaoxian.h5.service.collage.CollageOrderCreateThread.run(CollageOrderCreateThread.java:19)
        at java.lang.Thread.run(Thread.java:748)
"MQ-AsyncArrayDispatcher-Thread-ffbaed55-2e5b-41d8-b7ca-7085e13768231" #177 daemon prio=5 os_prio=0 tid=0x00007f9494ef2800 nid=0xcbd runnable [0x00007f93ed4d5000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x0000000680de4228> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
        at java.util.concurrent.ArrayBlockingQueue.poll(ArrayBlockingQueue.java:418)
        at com.alibaba.ons.open.trace.core.dispatch.impl.AsyncArrayDispatcher$AsyncRunnable.run(AsyncArrayDispatcher.java:200)
        at java.lang.Thread.run(Thread.java:748)
"NettyClientWorkerThread_4" #169 prio=5 os_prio=0 tid=0x00007f93f8026800 nid=0xcaf waiting on condition [0x00007f93edadb000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x0000000680e0c8e8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
        at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:467)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.SingleThreadEventExecutor.takeTask(SingleThreadEventExecutor.java:269)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.DefaultEventExecutor.run(DefaultEventExecutor.java:39)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:140)
        at java.lang.Thread.run(Thread.java:748)
"NettyClientWorkerThread_3" #168 prio=5 os_prio=0 tid=0x00007f93f8025000 nid=0xcad waiting on condition [0x00007f93edbdc000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x0000000680e105c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
        at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:467)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.SingleThreadEventExecutor.takeTask(SingleThreadEventExecutor.java:269)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.DefaultEventExecutor.run(DefaultEventExecutor.java:39)
        at com.aliyun.openservices.shade.io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:140)
        at java.lang.Thread.run(Thread.java:748)
```

也可直接查看相关信息
```
-bash-4.2$ jstack pid|grep tid十六进制 -A 20
"Thread-32" #209 prio=5 os_prio=0 tid=0x00007f2baa580800 nid=0xa78 runnable [0x00007f2afeae0000]
   java.lang.Thread.State: RUNNABLE
    at java.lang.Throwable.fillInStackTrace(Native Method)
    at java.lang.Throwable.fillInStackTrace(Throwable.java:783)
    - locked <0x000000077268f118> (a redis.clients.jedis.exceptions.JedisConnectionException)
    at java.lang.Throwable.<init>(Throwable.java:287)
    at java.lang.Exception.<init>(Exception.java:84)
    at java.lang.RuntimeException.<init>(RuntimeException.java:80)
    at redis.clients.jedis.exceptions.JedisException.<init>(JedisException.java:15)
    at redis.clients.jedis.exceptions.JedisConnectionException.<init>(JedisConnectionException.java:15)
    at redis.clients.util.Pool.getResource(Pool.java:50)
    at redis.clients.jedis.JedisPool.getResource(JedisPool.java:99)
    at com.jfinal.plugin.redis.Cache.getJedis(Cache.java:1253)
    at com.woniubaoxian.common.utils.BnailRedisUtil.llen(BnailRedisUtil.java:105)
    at com.woniubaoxian.h5.service.collage.CollageOrderCreateThread.run(CollageOrderCreateThread.java:19)
    at java.lang.Thread.run(Thread.java:748)
```
查看完毕后，将进程用户修改回无法终端登陆
```
usermod -s /sbin/nologin jetty
chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow
```
