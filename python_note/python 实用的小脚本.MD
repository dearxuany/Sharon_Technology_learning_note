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
