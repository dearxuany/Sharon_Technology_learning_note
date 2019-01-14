# Python 通过 yagmail 发送邮件
通过python第三方库 yagmail 发邮件，会比使用标准库 smtplib 简单很多。
```
#! /usr/bin/python3

import yagmail

# 链接邮箱服务器以及发件邮箱信息
yag=yagmail.SMTP(user='username@163.com',password='***',host='smtp.163.com')

# 邮件内容
contents=['Testing the use of python3 yagmail!']

# 收件邮箱地址、标题、内容
yag.send('username@qq.com','python3 yagmail test',contents)

```
多个收件人修改最后一行
```
yag.send(['email_add1,email_add2,emailadd3'],'Subject'，'contents')
```
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Python_note_images/python3%20yagmail.jpg)

参考</br>
http://www.cnblogs.com/fnng/p/7967213.html </br>
网易邮箱支持POP3/SMTP/IMAP服务</br>
http://help.163.com/09/1223/14/5R7P3QI100753VB8.html </br>
