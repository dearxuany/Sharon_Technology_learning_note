# Python 日期与时间
日期与时间相关模块：</br>
* time 时间相关操作，通常用于格式化时间；</br>
* calendar 日期相关操作，通常用于格式化日期；</br>
* datatime 提供了日期和时间的算术支持；</br>
* timeit 用于计算代码的执行时间。</br>

## time
模块time：</br>
1.获取当前时间日期；</br>
2.获取操作时间和日期；</br>
3.从字符串读取时间日期；</br>
4.改变日期、时间的表示格式。</br>
</br>
新纪元：</br>
根据操作系统而异，UNIX系统认为1970年1月1日0点是时间纪元。时间戳单位最适于做日期运算，太遥远的日期也不行，UNIX和Windows32位只支持到2038年。</br>
```
>>> import time
>>> dir(time)
['CLOCK_MONOTONIC', 'CLOCK_MONOTONIC_RAW', 'CLOCK_PROCESS_CPUTIME_ID', 'CLOCK_REALTIME', 'CLOCK_THREAD_CPUTIME_ID', '_STRUCT_TM_ITEMS', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'altzone', 'asctime', 'clock', 'clock_getres', 'clock_gettime', 'clock_settime', 'ctime', 'daylight', 'get_clock_info', 'gmtime', 'localtime', 'mktime', 'monotonic', 'perf_counter', 'process_time', 'sleep', 'strftime', 'strptime', 'struct_time', 'time', 'timezone', 'tzname', 'tzset']
```
### 按照固定格式获取当前时间（最简单）
```
>>> time.asctime()
'Wed Aug 29 10:55:11 2018'
```
### 时间戳
时间戳：</br>
如果计算从某一日到现在的相对秒数，则linux默认是从“新纪元”处的时刻计算到现时所在的时刻，这个计算所得结果（单位：秒）被称为“时间戳”。</br>
```
# 当前时间的时间戳，从“新纪元”开始后的秒数，以UTC为准
>>> time.time()
1535509721.5329
```

### 时间元组
时间元组 struct_time ：</br>
9位时间元组，0-8位置分别表示“年、月、日、时、分、秒、星期、儒略日、夏令时”。</br>
秒的范围为0-61，因为有闰一秒和闰两秒；</br>
星期由0-6表示，星期一为0；</br>
儒略日，一年的第几日，范围0-366，传统上的儒略日是一个很长的数字，通常不会用到；</br>
夏令时，-1是决定是否为夏令时的旗帜，当为-1时，mktime得到的值可能正确。</br>

#### 时间戳转时间元组
```
>>> type(time.time())
<class 'float'>

# 转为本地时间的时间元组，如果想转为国际标准时间用gmtime
>>> time.localtime(time.time())
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=29, tm_hour=11, tm_min=24, tm_sec=15, tm_wday=2, tm_yday=241, tm_isdst=0)

# 对于当前时间来说，其实没有参数也是可以的
>>> time.localtime()
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=29, tm_hour=11, tm_min=26, tm_sec=3, tm_wday=2, tm_yday=241, tm_isdst=0)

# 特定时间戳转时间元组
>>> time.localtime(1381419600)
time.struct_time(tm_year=2013, tm_mon=10, tm_mday=10, tm_hour=23, tm_min=40, tm_sec=0, tm_wday=3, tm_yday=283, tm_isdst=0)
```
#### 时间元组转时间戳
```
>>> time.mktime( time.localtime())   # mktime和localtime作用相反
1535510542.0

# 特定时间元组转时间戳
>>> timeArray
time.struct_time(tm_year=2013, tm_mon=10, tm_mday=10, tm_hour=23, tm_min=40, tm_sec=0, tm_wday=3, tm_yday=283, tm_isdst=0)
>>> time.mktime(timeArray)
1381419600.0
```
### 格式化时间输出
```
# 时间元组格式化为字符串
>>> time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
'2018-08-29 11:53:37'

# 格式化字符串转时间元组
>>> a = "Wed Aug 29 10:55:11 2018"
>>> time.strptime(a,"%a %b %d %H:%M:%S %Y")
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=29, tm_hour=10, tm_min=55, tm_sec=11, tm_wday=2, tm_yday=241, tm_isdst=-1)
```
python中时间日期格式化符号：
```
%y 两位数的年份表示（00-99）
%Y 四位数的年份表示（000-9999）
%m 月份（01-12）
%d 月内中的一天（0-31）
%H 24小时制小时数（0-23）
%I 12小时制小时数（01-12）
%M 分钟数（00=59）
%S 秒（00-59）
%a 本地简化星期名称
%A 本地完整星期名称
%b 本地简化的月份名称
%B 本地完整的月份名称
%c 本地相应的日期表示和时间表示
%j 年内的一天（001-366）
%p 本地A.M.或P.M.的等价符
%U 一年中的星期数（00-53）星期天为星期的开始
%w 星期（0-6），星期天为星期的开始
%W 一年中的星期数（00-53）星期一为星期的开始
%x 本地相应的日期表示
%X 本地相应的时间表示
%Z 当前时区的名称
%% %号本身
```

### 休眠
time.sleep(secs)：用于推迟调用线程的运行，secs指秒数。
```
#! usr/bin/python3

import time

print('now：',time.asctime())
time.sleep(10)   # 等待10秒
print('10 seconds later and now :',time.asctime())

# 输出
now： Wed Aug 29 12:15:11 2018
10 seconds later and now : Wed Aug 29 12:15:21 2018
```

## 拓展
### UTC
协调世界时(Universal Time Coordinated)，协调世界时是以原子时秒长为基础，在时刻上尽量接近于世界时的一种时间计量系统。为确保协调世界时与世界时相差不会超过0.9秒，在有需要的情况下会在协调世界时内加上正或负闰秒。因此协调世界时与国际原子时之间会出现若干整数秒的差别，两者之差逐年积累，便采用跳秒（闰秒）的方法使协调时与世界时的时刻相接近，其差不超过1s。它既保持时间尺度的均匀性，又能近似地反映地球自转的变化。这套时间系统被应用于许多互联网和万维网的标准中，例如，网络时间协议就是协调世界时在互联网中使用的一种方式。

### 闰一秒
1972年至今，一共出现了24个闰秒，原因是地球自转速度变慢。因为地球自转不规则，闰秒没有规律可循，一般会由天文台提前半年通知。一般会在每年的6月30日、12月31日的最后一秒进行调整。

### 儒略日
儒略日的起点订在公元前4713年（天文学上记为 -4712年）1月1日格林威治时间平午（世界时12:00），即JD 0指定为UT时间B.C.4713年1月1日12:00到UT时间B.C.4713年1月2日12:00的24小时。每一天赋予了一个唯一的数字，顺数而下，如：1996年1月1日12:00:00的儒略日是2450084。这个日期是考虑了太阳、月亮的轨道运行周期，以及当时收税的间隔而订出来的。

### 关于时间戳最大支持到2038年的问题
Unix时间戳（英文为Unix epoch, Unix time, POSIX time 或 Unix timestamp）是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数，不考虑闰秒。 </br>
在大部份的32位操作系统上，此“time_t”数据模式使用一个有正负号的32位元整数(signedint32)存储计算的秒数。依照此“time_t”标准，在此格式能被表示的最后时间是2038年1月19日03:14:07，星期二（UTC）。超过此一瞬间，时间将会被掩盖（wrap around）且在内部被表示为一个负数，并造成程序无法工作，因为它们无法将此时间识别为2038年，而可能会依个别操作而跳回1970年或1901年。错误的计算及动作可能因此产生。

