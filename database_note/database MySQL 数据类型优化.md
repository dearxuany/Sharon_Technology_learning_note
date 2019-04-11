# MySQL 数据类型优化
## 数据类型
### 选择原则
* 选择不会超过范围的最小类型</br>
占更少资源、CPU周期更少（整形比字符代价更低）
* 使用简单的数据类型</br>
使用mysql内建类型存储日期和时间，用整形存IP
* 通常情况下最好指定列为 NOT NULL，除非真的需要存 NULL 值
查询中包含NULL的列，MySQL更难优化；</br>
NULL列会花更多的存储空间，NULL列被索引时，每个索引记录需要一个额外字节；</br>
如果要在NULL列上建索引，应尽量避免设计成可NULL列；</br>
InnoDB使用单独的为(bit)存储NULL值，可用于存储稀疏数据（很多行为NULL，少数行为非NULL）</br>

### 选择具体类型
因素：存储长度和范围、精度、磁盘空间、内存空间、特殊行为和属性</br>
http://naotu.baidu.com/file/f2b4c4ff2fe54e9202b34a1beb14fe79</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/database_note_images/mysql%2B%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)

