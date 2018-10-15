# Python使用CGI创建动态网页 cgi、cgitb
CGI 通用网关接口 Common Gateway Interface ：</br>
CGI 是Web 服务器运行时外部程序的规范，按CGI 编写的程序可以扩展服务器功能。CGI是一种创建web应用的简单方式，CGI规范允许Web服务器执行外部程序，并将它们的输出发送给Web浏览器，CGI将Web的一组简单的静态超媒体文档变成一个完整的新的交互式媒体。CGI 应用程序能与浏览器进行交互，还可通过数据库API 与数据库服务器等外部数据源进行通信，从数据库服务器中获取数据。几乎所有服务器都支持CGI,可用任何语言编写CGI。
Python 中和cgi编程有关的模块是cgi和cgitb。</br>
当然不直接使用CGI也可使用现成的python Web框架，如Flask和Django等。</br>


## 创建CGI运行目录
所有的HTTP服务器执行CGI程序都保存在一个预先配置的目录，目录位置要根据服务器的选用而定。</br>
默认情况下，Linux服务器配置运行的cgi-bin目录中为/var/www。</br>
此处只尝试使用CGI，所以使用python中的http.server模块直接运行一个临时的Web服务器，使用子目录/cgi-bin。</br>

## Web服务器支持与启动
到目标目录cgi-bin下启动服务器并指定支持CGI</br>
注意：要注意路径，该目录内的所有文件都被服务器公开，注意文件安全</br>
```
$ python3 -m http.server --cgi
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
用浏览器打开该地址，此处设置的本地 127.0.0.1:8000 ，现时该目录里面是没有东西的</br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Python_note_images/python%20httpserver.png)
```
127.0.0.1 - - [15/Oct/2018 15:32:26] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [15/Oct/2018 15:32:27] code 404, message File not found
127.0.0.1 - - [15/Oct/2018 15:32:27] "GET /favicon.ico HTTP/1.1" 404 -
127.0.0.1 - - [15/Oct/2018 15:32:28] code 404, message File not found
127.0.0.1 - - [15/Oct/2018 15:32:28] "GET /favicon.ico HTTP/1.1" 404 -
```

## CGI脚本
在目标目录中，创建简单的CGI脚本：</br>
CGI文件的扩展名为.cgi，因为服务器用的是python的模块，所以脚本可用.py扩展名。</br>
注意：这个脚本必须放在./cgi-bin里面！！然后服务器在此目录的上一层目录运行！！要注意/cgi-bin的权限</br>
```
#! /usr/bin/python3

print('Content-type: text/html\n') # 类型声明 

print('Hello World!')
```
修改脚本权限
```
# chmod 755 simple1.py
```
启动服务器进入本机地址，可以看到脚本已经在里面了

浏览器运行 http://127.0.0.1:8000/cgi-bin/simple1.py </br>
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Python_note_images/python%20CGI.png)

## 使用cgitb进行调试
修改上面的脚本为以下内容可启用CGI栈跟踪，以便调试。</br>
注意：仅作开发调试用！注意文件权限</br>
```
#! /usr/bin/python3

import cgitb

cgitb.enable()  # 启用CGI栈跟踪

print('Content-type: text/html\n') # 类型声明 
print(1/0)  # 错误语句

print('Hello World!')
```
![](https://github.com/dearxuany/Sharon_Technology_learning_note/blob/master/note_images/Python_note_images/python%20cgitb.png)</br>
因为有中文注释，所以如有必要要注意转码
