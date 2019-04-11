# MySQL InnoDB锁机制
InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁。</br>
参考：https://www.cnblogs.com/aipiaoborensheng/p/5767459.html</br>
## 行锁模式及加锁方法
如果一个事务请求的锁模式与当前的锁兼容，InnoDB就将请求的锁授予该事务；</br>
反之，如果两者不兼容，该事务就要等待锁释放。</br>
http://naotu.baidu.com/file/02366c3ff486e3bd87be3a3e695723db </br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/database_note_images/innodb%2B%E9%94%81%E6%9C%BA%E5%88%B6%20(1).png)
