# python 实用的小脚本
## 将字节改为如KB/MB/GB等其他单位
M=million，K=kilo，B=byte，b=bit</br>
1B=1byte=8bit，1KB=1024B，1MB=1024KB，1GB=1024MB，1T=1024GB</br>
```
>>> symbols = ('K','M','G','T','P','E','Z','Y')
>>>
>>> enumerate(symbols)
<enumerate object at 0xb6f4f414>
>>> list(enumerate(symbols))
[(0, 'K'), (1, 'M'), (2, 'G'), (3, 'T'), (4, 'P'), (5, 'E'), (6, 'Z'), (7, 'Y')]
>>> prefix = {}
>>> for i,s in enumerate(symbols):
...         prefix[s]= 1 << (i+1)*10
... 
>>> print(prefix)
{'K': 1024, 'M': 1048576, 'G': 1073741824, 'T': 1099511627776, 'P': 1125899906842624, 'E': 1152921504606846976, 'Z': 1180591620717411303424, 'Y': 1208925819614629174706176}
>>> 

```
注意：</br>
二进制 >>(左移) <<(右移)</br>
例如 10 的二进制表示 1000，右移两位 10 << 2 的值就是 40，40二进制表示为 100000</br>
```
def change_measure_unit(n):
    symbols = ('K','M','G','T','P','E','Z','Y')
    prefix = {}
    # 给单位赋值如K的值为1024字节存在prefix字典中
    for i,s in enumerate(symbols):
        prefix[s]= 1 << (i+1)*10  # 二进值右移
    # reverse是为了减少循环次数，不需要由小到大一个个判断
    for s in reversed(symbols):
        if n >= prefix[s]:
            value = float(n)/prefix[s]
            return '{:.2f}{}B'.format(value,s)


>>> print(change_measure_unit(1058263040))
1009.24MB


```
## 批量生成名称中含有日期的文件或目录
```
#! /usr/bin/env python3

import os

def make_date_format():
    mounth_list = list(range(1,13))
    day_list = list(range(1,32))

    # 修改列表内容的类型并在个位前面加0
    for n in range(12):
        mounth_list[n] = str(mounth_list[n])
        if n <= 8:
            mounth_list[n] = '0'+mounth_list[n]

    for n in range(31):
        day_list[n] = str(day_list[n])
        if n <= 8:
            day_list[n] = '0'+day_list[n]

    # 生成一年内每天的日期
    date_format = []
    for m in range(12):
        if m == 1:
            for n in range(29):
                date_format.append(mounth_list[m]+day_list[n])
        elif m == 3 or m == 5 or m == 8 or m == 10:
            for n in range(30):
                date_format.append(mounth_list[m]+day_list[n])
        else:
            for n in range(31):
                date_format.append(mounth_list[m]+day_list[n])
    return date_format

def make_files(date_format):
    # 执行命令批量生成文件或目录
    for n in range(len(date_format)):
        filename_format = '2019'+date_format[n]+'_backup_logs'
        command = 'mkdir'+' '
        os.system(command + filename_format)

if __name__ == '__main__':
    make_files(make_date_format())
```
