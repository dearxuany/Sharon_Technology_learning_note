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

## 
