# shell 管道命令
作用</br>
* 选取命令，在大量信息中筛选有用信息，不会对文本本身进行修改；</br>
* 文本处理，直接通过管道命令对文本内容进行修改。</br>

条件</br>
仅处理stdout信息，处理的命令必须支持stdin，否则需要用 xargs 作处理</br>

常见管道命令</br>
* 选取命令 cut grep egrep</br>
* 排序统计命令 sort wc uniq</br>
* 双重定向 tee</br>
* 字符转换 tr col join paste expand</br>
* 参数代换 xargs</br>
* 文本处理 sed awk</br>

涉及特殊字符 |

## 选取命令
### cut
cut 按要求的关键字符分割列信息，以取出数据中的某些“列”</br>
可以将信息看做关键字符分割的表格，然后按需提取出来</br>
cut 只筛选数据，不对数据本身做修改</br>
```
cut -d ‘分隔字符’ -f 要取出的列的编号（可用,隔开）
cut -c 12-20（要分隔的字符区间）
```
```
$ echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/src/node-v8.11.3-linux-x86/bin:/home/sunnylinux/.local/bin:/home/sunnylinux/bin

#以：作为分隔，取第二块
$ echo $PATH | cut -d : -f 2
/usr/local/sbin
$ echo $PATH | cut -d : -f 2,5
/usr/local/sbin:/bin

#分隔字符
$ echo $PATH | cut -c 16-32
/usr/local/sbin:/
```
用 cut 来分析 last，last 用来列出目前与过去登录系统的用户相关信息，指令英文原义：show listing of last logged in users
```
$ last
sunnylin pts/2        :0               Fri Dec 21 15:52 - 15:52  (00:00)    
sunnylin pts/1        :0               Fri Dec 21 15:52 - 15:52  (00:00)    
sunnylin pts/0        :0               Fri Dec 21 15:52   still logged in   
sunnylin :0           :0               Fri Dec 21 15:45   still logged in   
reboot   system boot  3.10.0-862.2.3.e Fri Dec 21 15:29 - 10:09  (18:39)    
sunnylin tty2                          Fri Dec 21 15:27 - 15:28  (00:01)    
sunnylin pts/1        :0               Thu Dec 20 11:20 - 11:21  (00:00)    
sunnylin pts/0        :0               Thu Dec 20 11:20 - 15:28 (1+04:07)   
sunnylin :0           :0               Thu Dec 20 11:04 - crash (1+04:24)   
reboot   system boot  3.10.0-862.2.3.e Thu Dec 20 10:45 - 10:09 (1+23:23)   

```
last 输出字段：</br>
第一列：用户名</br>
第二列：终端位置，pts/0 (伪终端) 意味着从诸如SSH或telnet的远程连接的用户.tty (teletypewriter) 意味着直接连接到计算机或者本地连接的用户</br>
第三列：登录ip或者内核，如果你看见:0.0 或者什么都没有，这意味着用户通过本地终端连接。除了重启活动，内核版本会显示在状态中</br>
第四列：开始时间</br>
第五列：结束时间（still login in 还未退出 down 直到正常关机 crash 直到强制关机）</br>
第六列：持续时间</br>
```
#仅输出用户名和终端
$ last|cut -d ' ' -f 1,2
sunnylin pts/2
sunnylin pts/1
sunnylin pts/0
sunnylin :0
reboot 
sunnylin tty2
sunnylin pts/1
sunnylin pts/0
sunnylin :0
reboot 
```
注意：</br>
cut 在处理多空格时不好使，如reboot后面有好多个空格就分不到第二块的数据了</br>
真要分析last的输出，建议使用last自带的参数</br>

### grep
grep 根据关键字筛选出某些行，常与正则表达式组合使用</br>
grep 查找到目标字符串时，会输出字符串所在的整一行数据</br>
grep 可直接处理文件中的信息，也可以作为管道命令使用</br>
```
#通过管道对stdout的信息进行处理
处理对象 |  grep [-acinv] ‘关键字’

#直接对文件进行处理
grep [-acinv] ‘关键字’ filename

-a 将binary以text的方式查找数据
-c 计算查找到‘关键字’的次数
-i 忽略大小写
-n 输出行号
-v 反向选择
```
#### grep 管道命令用法
仅选出和root有关的登录信息
```
$ last | grep root
root     tty2                          Mon Oct  1 17:28 - 17:29  (00:01)    
root     pts/1        :1               Wed Aug  8 20:16 - 20:21  (00:05)    
root     pts/1        :1               Wed Aug  8 20:10 - 20:16  (00:05)    
root     :1           :1               Wed Aug  8 18:36 - crash  (09:47)    
root     tty2                          Tue Aug  7 16:31 - 16:32  (00:01)    
root     tty2                          Fri Jul 20 18:01 - 18:02  (00:00)    
root     tty2                          Fri May 18 12:50 - 12:51  (00:00)   
(省略很多输出)
```
反选
```
$ last | grep -v root
sunnylin pts/2        :0               Fri Dec 21 15:52 - 15:52  (00:00)    
sunnylin pts/1        :0               Fri Dec 21 15:52 - 15:52  (00:00)    
sunnylin pts/0        :0               Fri Dec 21 15:52   still logged in   
sunnylin :0           :0               Fri Dec 21 15:45   still logged in   
reboot   system boot  3.10.0-862.2.3.e Fri Dec 21 15:29 - 11:37  (20:07)    
sunnylin tty2                          Fri Dec 21 15:27 - 15:28  (00:01)    
```
查看特定进程信息
```
$ ps -aux|grep python3
sunnyli+  4804  0.5  0.4  11400  4448 pts/0    T    11:39   0:00 python3
sunnyli+  4827  0.0  0.0   6704   884 pts/0    R+   11:40   0:00 grep --color=auto python3
```
#### grep 对文件内容进行筛选
linux和macOS 标准字典文件：/usr/share/dict/words </br>
https://en.wikipedia.org/wiki/Words_(Unix) </br>
</br>
查找含有特定的字符的行并输出行号
```
$ cat /usr/share/dict/words|grep -n 'apple'
22097:apple
22098:appleberry
22099:appleblossom
22101:applecart
22102:apple-cheeked
22103:appled
22105:appledrane
22106:appledrone
22107:apple-eating
22108:apple-faced
22109:apple-fallow
22111:applegrower
22112:applejack
22113:applejacks
（省略很多输出）

#不用管道命令的写法，输出同上
$ grep -n 'apple' /usr/share/dict/words
```

### egrep
egrep和grep功能基本相似，除了支持基础正则表达式外，还支持扩展正则表达式，以实现更谨慎的筛选。
```
$ ps aux | egrep '(python3|cron)'
root      1223  0.0  0.0   7984   680 ?        Ss   14:47   0:01 /usr/sbin/crond -n
sunnyli+  5039  0.0  0.4  11400  4552 pts/0    T    21:49   0:00 python3
sunnyli+  5429  0.0  0.0   6708   892 pts/0    R+   22:58   0:00 grep -E --color=auto (python3|cron)
```
## 排序统计命令
### sort
sort 可根据数据的类型来对数据进行排序，可是也要注意语系</br>
和grep一样，可作为管道命令，也可直接对文件中的数据进行操作</br>
sort 不改变文件内容本身</br>
```
sort [-fbMnrtuk] filename
排序对象 | sort [-fbMnrtuk]

默认按字符串排序a-z
-f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
-b  ：忽略最前面的空格符部分；
-M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
-n  ：使用『纯数字』进行排序(预设是以文字型态来排序的)；
-r  ：反向排序；
-u  ：就是 uniq ，相同的资料中，仅出现一行代表；
-t  ：分隔符，预设是用 [tab] 键来分隔；
-k  ：以那个区间 (field) 来进行排序的意思
```

### uniq
uniq 和sort中-u功能相同，可是有个参数-c，可用于计算相同筛选结果的重复次数</br>
```
#计算每位用户的登录次数
$ last|cut -d ' ' -f 1|sort|uniq -c
      1 
    102 reboot
     39 root
    238 sunnylin
      1 wtmp
```
注意：使用uniq计数前，必须先用sort排序，否则会出现计数错误

### wc
wc 常用于文件字数、行数、字符数的统计
```
wc [-lwm]
默认输出为 行数、字数、字符数
-l 行数
-w 英文词数
-m 字符数
```
三个输出分别是 行数、词数、字符数
```
$ cat /usr/share/dict/words|wc
 479828  479828 4953680
```
计算月登录系统的人数
```
#选出8月有关的数据，然后筛掉reboot的数据
last|grep 'Aug'|grep -v 'reboot'
sunnylin pts/0        :0               Thu Aug 30 14:27 - 13:13 (3+22:46)   
sunnylin :0           :0               Thu Aug 30 14:25 - 13:13 (3+22:47)   
sunnylin pts/0        :0               Thu Aug 30 12:15 - 14:10  (01:54)    
sunnylin :0           :0               Thu Aug 30 09:38 - down   (04:33)    
sunnylin pts/1        :0               Wed Aug 29 10:28 - 10:28  (00:00)    
sunnylin pts/0        :0               Wed Aug 29 09:50 - 09:34  (23:44)    
（省略部分输出）

#统计出8月的登录人数
$ last|grep 'Aug'|grep -v 'reboot'|wc -l
43
```
计算该月各用户登录次数
```
$ last|grep 'Aug'|grep -v 'reboot'|cut -d ' ' -f 1|sort|uniq -c
      4 root
     39 sunnylin
```

## 双向重定向
### tee
数据流重定向用于将stdout输出到文件，而管道命令用于将stdout输出到屏幕</br>
tee 可用于把stdout输出到文件同时输出到屏幕，但要注意输出和筛选顺序，会影响结果</br>
```
tee [-a] filename
-a  append以累加方式添加数据
```
将某月的登录数据输出到文件并进行该月的用户登录统计
```
$ last|grep 'Aug'|grep -v 'reboot'|tee last_list_aug|cut -d ' ' -f 1|sort|uniq -c
      4 root
     39 sunnylin

$ ls -l ./last_list_aug 
-rw-rw-r--. 1 sunnylinux sunnylinux 3311 12月 22 12:58 ./last_list_aug
$ cat ./last_list_aug 
sunnylin pts/0        :0               Thu Aug 30 14:27 - 13:13 (3+22:46)   
sunnylin :0           :0               Thu Aug 30 14:25 - 13:13 (3+22:47)   
sunnylin pts/0        :0               Thu Aug 30 12:15 - 14:10  (01:54)    
sunnylin :0           :0               Thu Aug 30 09:38 - down   (04:33)    
sunnylin pts/1        :0               Wed Aug 29 10:28 - 10:28  (00:00) 
```



