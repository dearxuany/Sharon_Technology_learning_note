# shell 管道命令
作用</br>
* 选取命令，在大量信息中筛选有用信息，不会对文本本身进行修改；</br>
* 文本处理，直接通过管道命令对文本内容进行修改。</br>

条件</br>
仅处理stdout信息，处理的命令必须支持stdin，否则需要用 xargs 作处理</br>

常见管道命令</br>
* 选取命令 cut grep egrep</br>
* 排序命令 sort wc uniq</br>
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
#直接对文件进行处理
grep [-acinv] ‘关键字’ filename

#通过管道对stdout的信息进行处理
处理对象 |  grep [-acinv] ‘关键字’

-a 将binary以text的方式查找数据
-c 计算查找到‘关键字’的次数
-i 忽略大小写
-n 输出行号
-v 反向选择
```
